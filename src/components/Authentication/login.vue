<script setup lang="js">
import AuthWrapper from "./AuthWrapper.vue"
import { ref } from "vue";
import { useRouter } from 'vue-router'
import { useAlertsStore } from "@/stores";
import {validations} from "@/utils/validation"
import userSvc from "@/services/userSvc";
import { setSessionStItem } from "@/utils/SessionStorage";
const AlertStore=useAlertsStore()
const router = useRouter()
const loginModel = ref({
  user_name: null,
  password: null,
});
var showPassword=ref(false)
const loginForm = ref();
 const login=async()=>{
  const validate=await loginForm.value?.validate()
  if(validate.valid){
    try {
      const responseData=await userSvc.Login(loginModel.value)
      if(responseData.data.status_code==200){
        AlertStore.showSuccessAlert(responseData.data.message)
        setSessionStItem("AuthData",responseData.data.data)
        getuserProfile(responseData.data.data[0].id)
        
      }else{
        AlertStore.showErrorAlert(responseData.data.message)
      }
      
    } catch (error) {
      console.log(error);
      AlertStore.showErrorAlert("Something Went wrong")
    }
    
  }
   
  
}
const getuserProfile=async(id)=>{
    try {
      const responseData=await userSvc.userProfile(id)
      if(responseData.data.status_code==200){
        setSessionStItem("is_auth",true)
        router.push("/home")
        setSessionStItem("user_details",responseData.data.data)
      }

    } catch (error) {
      console.log(error);
    }
}
</script>

<template>
  <AuthWrapper>
    <v-card-text>
      <p class="text-center">
        <h3 class="font-weight-bold">
          Welcome to Anand's Jewelry.
        </h3>
        Unlock treasures within.
        Login to your glittering journey.
      </p>
    <v-form ref="loginForm">
      <v-text-field
        v-model="loginModel.user_name"
        color="primary"
        label="Email"
        :rules="validations.emailValidations"
        placeholder="Enter your Email"
      ></v-text-field>
      <v-text-field
        v-model="loginModel.password"
        color="primary"
        label="Password"
        :append-inner-icon="showPassword ? 'fa-eye' : 'fa-eye'"
        :type="showPassword ? 'text' : 'password'"
        @click:append-inner="showPassword= !showPassword"
        :rules="validations.passwordValidations"
        placeholder="Enter your password"
      ></v-text-field>
      <v-card-actions>
        <v-spacer></v-spacer>

        <v-btn block @click="login()">
          Login
          <v-icon icon="fas fa-caret-right" end></v-icon>
        </v-btn>
      </v-card-actions>
    </v-form>
      <v-divider></v-divider>
      <p class="text-center">
        Have you frogot your password? click
        <RouterLink to="/forgot-password">Forgot Password</RouterLink> to reset.
      </p>
    </v-card-text>
  </AuthWrapper>
</template>

<style scoped></style>
