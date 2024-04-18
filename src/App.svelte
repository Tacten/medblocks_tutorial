<script lang="ts">
  import { onMount } from 'svelte';
  import { CLIENT_ID, FHIR_BASE_URL, REDIRECT_URI, SMART_AUTH_URL, SMART_TOKEN_URL} from '../config';
  import axios from 'axios';
  import pkceChallenge from "pkce-challenge";


  const generateCodeChallenge = async() => {
    const { code_challenge, code_verifier } = await pkceChallenge();
    // console.log(code_challenge,code_verifier)
    localStorage.setItem('code_verifier', code_verifier);
    return code_challenge;
  };

  const generateRedirectUrl = (codeChallenge:string) => {
    const authUrl = new URL(SMART_AUTH_URL);
    authUrl.searchParams.set('client_id',CLIENT_ID);
    authUrl.searchParams.set('scope','openid fhirUser')
    authUrl.searchParams.set('redirect_uri',REDIRECT_URI);
    authUrl.searchParams.set('response_type','code');
    authUrl.searchParams.set('state','1234567');
    authUrl.searchParams.set('aud',FHIR_BASE_URL);
    authUrl.searchParams.set('code_challenge',codeChallenge);
    authUrl.searchParams.set('code_challenge_method','S256');

    return authUrl.href
  };

  const makeTokenRequest =async(code:string , codeVerifier:string )=>{
    const tokenRequestForm = new FormData()
    tokenRequestForm.set('grant_type','authorization_code')
    tokenRequestForm.set('code',code)
    tokenRequestForm.set('redirect_uri',REDIRECT_URI)
    tokenRequestForm.set('client_id',CLIENT_ID)   
    tokenRequestForm.set('code_verifier',codeVerifier)
    
    const response = await axios.postForm(SMART_TOKEN_URL,tokenRequestForm)
    console.log(response)
  }

  onMount(async()=>{
    const code = new URLSearchParams(window.location.search).get('code')
    const codeVerifier = localStorage.getItem('code_verifier')
    if(code  && codeVerifier){
      await makeTokenRequest(code,codeVerifier)
      localStorage.removeItem('code_verifier')
    }
  })

  const initiateAuthRequest = async()=>{
    const codeChallenge = await generateCodeChallenge()
    const redirectUrl = generateRedirectUrl(codeChallenge)
    console.log(redirectUrl)
  }


</script>
<div class="flex justify-center items-center h-screen w-screen">
  <button class="bg-blue-600 px-4 py-2 rounded-md text-white" on:click={initiateAuthRequest}>Sign in with Epic</button>
</div>