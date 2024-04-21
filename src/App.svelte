<script lang="ts">
  import { onMount } from 'svelte';
  import { CLIENT_ID, FHIR_BASE_URL, REDIRECT_URI, SMART_AUTH_URL, SMART_TOKEN_URL} from '../config';
  import axios from 'axios';
  import pkceChallenge from "pkce-challenge";
  import PatientViewer from './lib/PatientViewer.svelte';
  import MedicationDetails from './lib/MedicationDetails.svelte';

  let tokenResponse:{
    access_token:string,
    id_token:string,
    patient:string,
    scope:string,
    issued_At:number,
    expired_in:number
  }
  let loading = true
  const generateCodeChallenge = async() => {
    const { code_challenge, code_verifier } = await pkceChallenge();
    // console.log(code_challenge,code_verifier)
    localStorage.setItem('code_verifier', code_verifier);
    return code_challenge;
  };

  const generateRedirectUrl = (codeChallenge:string) => {
    const authUrl = new URL(SMART_AUTH_URL);
    // console.log(CLIENT_ID,REDIRECT_URI,FHIR_BASE_URL,codeChallenge,SMART_AUTH_URL,SMART_TOKEN_URL,"code challenge")
    authUrl.searchParams.set('client_id',CLIENT_ID);
    authUrl.searchParams.set('scope','openid fhirUser')
    authUrl.searchParams.set('redirect_uri',REDIRECT_URI);
    authUrl.searchParams.set('response_type','code');
    // authUrl.searchParams.set('state','1234567');
    authUrl.searchParams.set('aud',FHIR_BASE_URL);
      authUrl.searchParams.set('code_challenge',codeChallenge);
      authUrl.searchParams.set('code_challenge_method','S256');

    return authUrl.href 
  };

  const makeTokenRequest = async (code: string, codeVerifier: string) => {
    const tokenRequestForm = new FormData();
    tokenRequestForm.set('grant_type', 'authorization_code');
    tokenRequestForm.set('code', code);
    tokenRequestForm.set('redirect_uri', REDIRECT_URI);
    tokenRequestForm.set('client_id', CLIENT_ID);
    tokenRequestForm.set('code_verifier', codeVerifier);
    const tokenGeneratedAt = Math.round(new Date().getTime() / 1000);
    const response = await axios.postForm(SMART_TOKEN_URL, tokenRequestForm);
    console.log(response);
    tokenResponse = response.data;
    localStorage.setItem("smart_token_Storage", JSON.stringify({ ...tokenResponse, issued_At: tokenGeneratedAt }));
};

onMount(async () => {
  console.log("Component mounted"); // Check if the component is mounting
  const code = new URLSearchParams(window.location.search).get('code');
  const codeVerifier = localStorage.getItem('code_verifier');
  const tokenResponseString = localStorage.getItem('smart_token_Storage');
  try {
    if (tokenResponseString) {
      tokenResponse = JSON.parse(tokenResponseString);
      const { issued_At } = tokenResponse;
      const expired = issued_At + tokenResponse.expired_in < Math.round(new Date().getTime() / 1000);
      if (expired) {
        localStorage.removeItem('smart_token_Storage');
        tokenResponse = {
          access_token: '',
          id_token: '',
          patient: '',
          scope: '',
          issued_At: 0,
          expired_in: 0
        };
      }
    } else{
      if (code && codeVerifier) {
        await makeTokenRequest(code, codeVerifier);
        localStorage.removeItem('code_verifier');
      }
    }
  } catch (e) {
    console.error("Error in onMount:", e); // Log any errors that occur
  }
  loading = false;
});

  const initiateAuthRequest = async()=>{
    const codeChallenge = await generateCodeChallenge()
    console.log(codeChallenge,"code challenge")
    const redirectUrl = generateRedirectUrl(codeChallenge)
    console.log(redirectUrl)
    window.location.href=redirectUrl
  }


</script>
<main>
  {#if loading}
    <div class="flex justify-center items-center h-screen w-screen">
      <div class="animate-spin rounded-full h-32 w-32 border-t-2 border-b-2 border-blue-500"></div>
    </div>
    {:else}
    {#if tokenResponse}
      <PatientViewer access_token={tokenResponse.access_token} patientID={tokenResponse.patient}/>
      <MedicationDetails access_token={tokenResponse.access_token} patientID={tokenResponse.patient}/>
      {:else}
      <div class="flex justify-center items-center h-screen w-screen">
        <button class="bg-blue-600 px-4 py-2 rounded-md text-white" on:click={initiateAuthRequest}>Sign in with Epic</button>
      </div>
    {/if}
  {/if}
</main>