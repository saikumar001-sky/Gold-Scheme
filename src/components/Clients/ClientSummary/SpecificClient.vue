<template>
  <ViewsWrapper>
    <v-data-table :headers="headersData" :items="desserts">
      <template v-slot:top>
        <span class="float-right"> </span>

        <v-toolbar color="rgb(255 245 193)" flat class="pa-6">
          <div>
            <h3 class="font-weight-bold">{{ client_details.full_name }}</h3>
            <h3>mobile:{{ client_details.mobile_number }}</h3>
            <h3>Adhaar No:{{ client_details.adhaar_number }}</h3>
          </div>
          <v-spacer></v-spacer>
          <v-divider inset vertical></v-divider>
          <v-btn
            @click="openPay()"
            class="ms-3 me-2 float-right"
            color="success"
            variant="tonal"
          >
            <v-icon
              icon="fas fa-cash-register"
              class="me-2"
              size="small"
            ></v-icon>
            Pay
          </v-btn>
          <div>
            <h3>
              Total Amount Paid:
              <span class="font-weight-bold"
                >Rs.{{ client_details.total_amount }}</span
              >
            </h3>
            <h3>
              Gold Obtained:
              <span class="font-weight-bold"
                >{{ client_details.total_gold }}gms</span
              >
            </h3>
          </div>
        </v-toolbar>
      </template>
      <template v-slot:item.actions="{ item }">
        <v-btn @click="recieptdiag(item)" small color="success" variant="tonal">
          <v-icon icon="fas fa-download" class="me-2" size="small"></v-icon>
          Receipt
        </v-btn>
      </template>
    </v-data-table>
    <v-dialog v-model="paydialog" width="auto">
      <v-card max-width="600px">
        <v-toolbar color="rgb(255 245 193)" flat density="compact">
          <v-icon
            class="ms-2"
            size="small"
            title="Logout"
            icon="fas fa-cash-register"
          ></v-icon
          ><span class="ms-2 text-h6"
            >Add Payment for : {{ client_details.full_name }}</span
          >
        </v-toolbar>
        <v-card-text>
          <v-container>
            <v-form ref="payForm">
              <v-row>
                <v-col cols="12">
                  <v-text-field
                    v-model="editedItem.amount"
                    label="Amount(RS)"
                    type="number"
                    :rules="FieldRequired('Amount')"
                    placeholder="Enter Amount"
                  ></v-text-field>
                </v-col>
                <v-col cols="12">
                  <v-text-field
                    v-model="editedItem.rate"
                    label="Rate (/gm)"
                    type="number"
                    @input="calculateGold(editedItem.rate)"
                    :rules="FieldRequired('Rate')"
                    placeholder="Enter Rate"
                  ></v-text-field>
                </v-col>

                <v-col cols="12">
                  <v-text-field
                    v-model="editedItem.gold_gain"
                    label="Gold Gain (gm)"
                    type="number"
                    :rules="FieldRequired('Gold Gain')"
                    placeholder="Enter Gold Gain"
                  ></v-text-field>
                </v-col>

                <v-col cols="12">
                  <v-text-field
                    v-model="editedItem.date_of_payment"
                    type="date"
                    label="Date of Payment"
                    :rules="FieldRequired('Date of Payment')"
                    placeholder="Enter Date of Payment"
                  ></v-text-field>
                </v-col>
              </v-row>
            </v-form>
          </v-container>
        </v-card-text>
        <v-divider></v-divider>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="error"
            width="130px"
            class="text-black"
            variant="tonal"
            @click="close"
          >
            Cancel
          </v-btn>
          <v-btn width="130px" color="success" variant="tonal" @click="save">
            Save
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    <v-dialog v-model="recieptdialog" width="auto">
      <html2Pdf
        :client_details="client_details"
        :selected_item="selected_item"
      ></html2Pdf>
    </v-dialog>
  </ViewsWrapper>
</template>

<script setup lang="js">
  import ViewsWrapper from "@/views/ViewsWrapper.vue"
  import html2Pdf from "@/utils/ExportPdf/vue-html-pdf.vue";
  import {ref,onMounted} from "vue"
import { FieldRequired } from "@/utils/validation";
  import { useRoute } from "vue-router";
  import userSvc from "@/services/userSvc";
import { useAlertsStore } from "@/stores";
const client_details=ref({})
const paydialog=ref(false)
const payForm=ref()
const recieptdialog=ref(false)
const selected_item=ref({})
const AlertStore=useAlertsStore()
const router = useRoute()
  const headersData = ref([
    {
      title: 'Pay Id',
      align: 'start',
      sortable: false,
      key: 'payment_number',
    },
    { title: 'Amount Paid(RS)', key: 'amount' },
    { title: 'rate (/gm)', key: 'rate' },
    { title: 'Gold Obtained(gm)', key: 'gold' },
    { title: 'Date', key: 'date_of_payment' ,width:"300px"},
    { title: 'Actions', key: 'actions', sortable: false },
  ]);
  const editedItem = ref({
    "client_id": 0,
  "rate": 0,
  "amount": 0,
  "gold_gain": 0,
  "date_of_payment": null
    });

const defaultItem = ref({
  "client_id": 0,
  "rate": 0,
  "amount": 0,
  "gold_gain": 0,
  "date_of_payment": null
    });
  const desserts=ref([]);
  onMounted(()=>{
    getclientDetail()
    gettxnDetail()
})
const openPay=()=>{
paydialog.value=true
editedItem.value = { ...defaultItem.value };
}
const calculateGold=(val)=>{
  editedItem.value.gold_gain=(editedItem.value.amount/val).toFixed(2)
}
const recieptdiag=(item)=>{
  recieptdialog.value=true
selected_item.value=item
}
const close = () => {
  paydialog.value = false;
  editedItem.value = { ...defaultItem.value };
};
const save = async () => {
  const validate=await payForm.value?.validate()
  if(validate.valid){
    try {
      editedItem.value.client_id=router.query.id
      const responseData=await userSvc.Addpayment(editedItem.value)
      if(responseData.data.status_code==200){
        AlertStore.showSuccessAlert(responseData.data.message)
        close();
        gettxnDetail()
      }else{
        AlertStore.showErrorAlert(responseData.data.message)
      }

    } catch (error) {
      console.log(error);
      AlertStore.showErrorAlert("Something Went wrong")
    }

  }

};
  const getclientDetail=async()=>{
    try {
      const responseData=await userSvc.getclientDetail(router.query.client_id)
      if(responseData.data.status_code==200){
        client_details.value=responseData.data.data[0]
      }else{
        AlertStore.showErrorAlert(responseData.data.message)
      }

    } catch (error) {
      console.log(error);
    }
}
const gettxnDetail=async()=>{
    try {
      const responseData=await userSvc.gettxnDetail(router.query.client_id)
      if(responseData.data.status_code==200){
        desserts.value=responseData.data.data[0].payment_list
      }else{
        AlertStore.showErrorAlert(responseData.data.message)
      }

    } catch (error) {
      console.log(error);
    }
}
</script>
<style scoped></style>
