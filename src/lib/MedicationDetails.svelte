<script lang="ts">
    import type { OperationOutcome, Patient } from "fhir/r2";
    import { FHIR_BASE_URL } from "../../config";
    import axios from "axios";
  import type { Bundle, MedicationRequest } from "fhir/r4";

  
    export let patientID:string
    export let access_token:string
  
    const getMedicationDetails = async () => {
      try {
        const patientResponse = await axios.get<Bundle<MedicationRequest>>( 
          `${FHIR_BASE_URL}/MedicationRequest/`,
          {
            params: {
                subject: patientID
              },
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
     
    const getMedicationRequestEntries = (bundle:Bundle<MedicationRequest | OperationOutcome>)=>{
      if(!bundle?.entry){
        return []
      }
      return bundle.entry?.filter(entry=>entry?.resource?.resourceType === "MedicationRequest") as Bundle<MedicationRequest>[]
    }
  </script>
  
  {#await getMedicationDetails()}
  loading....
  {:then medicationlist}
    <div class="flex justify-center items-center flex-col mt-10">
      <h1>Medication List</h1>
        <div class="mx-32">
            <!-- <p>{JSON.stringify(medicationlist)}</p> -->
        </div>
        {#each getMedicationRequestEntries(medicationlist) as medication,i}
          <p>{i+1} {medication.resource?.medicationReference?.display}</p>
          <div>
            {#if medication?.resource?.reasonCode?.[0]?.text}
              Medication for {medication?.resource?.reasonCode?.[0]?.text}
            {/if}
            {#if medication?.resource?.dosageInstruction?.[0]?.text}
              Dosage: {medication?.resource?.dosageInstruction?.[0]?.text}
            {/if}
          </div>
        {/each}
    </div>
  {/await}
  