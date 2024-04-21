<script lang="ts">
    import type { Patient } from "fhir/r2";
    import { FHIR_BASE_URL } from "../../config";
    import axios from "axios";

  
    export let patientID:string
    export let access_token:string
  
    const getPatientDetails = async () => {
      try {
        const patientResponse = await axios.get<Patient>(
          `${FHIR_BASE_URL}/Patient/${patientID}`,
          {
            headers: {
              Authorization: `Bearer ${access_token}`
            }
          }
        );
        console.log(patientResponse.data); // Assuming patient data is in `data` property
        return patientResponse.data;
      } catch (error) {
        console.error("Error fetching patient details:", error);
        throw error; // Rethrow the error to handle it where this function is called
      }
    }
  </script>
  
  {#await getPatientDetails()}
  loading....
  {:then patient}
    <div class="h-full w-full flex justify-start mt-24 items-center flex-col">
        <div>
            <h1 class="font-semibold text-xl">Hello {patient.name[0].given[0]} {patient.name[0].family}</h1>
            <p class="">Welcome to you patient record<p>
        </div>
    </div>
  {/await}
  