<script lang="ts">
  import { CLIENT_ID, FHIR_BASE_URL, REDIRECT_URI, SMART_AUTH_URL} from '../config';

  const generateRedirectUrl = () => {
    const authUrl = new URL(SMART_AUTH_URL);
    authUrl.searchParams.set('client_id',CLIENT_ID);
    authUrl.searchParams.set('scope','openid fhirUser')
    authUrl.searchParams.set('redirect_uri',REDIRECT_URI);
    authUrl.searchParams.set('response_type','code');
    authUrl.searchParams.set('aud',FHIR_BASE_URL);
    authUrl.searchParams.set('state','1234567');
    // authUrl.searchParams.set('code_challenge ;
    return authUrl.href
  };

</script>
<div class="flex justify-center items-center h-screen w-screen">
  <button class="bg-blue-600 px-4 py-2 rounded-md text-white" on:click={()=>{console.log(generateRedirectUrl())}}>Sign in with Epic</button>
</div>