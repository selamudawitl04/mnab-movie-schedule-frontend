<!-- The header -->
<script setup>

import {useAuthStore} from '@/stores/modules/auth'
import updateUserMutation from '@/graphql/auth/updateUser.gql'
import mutation from '@/composables/mutation'
import { Form, Field } from 'vee-validate';
import * as Yup from 'yup';
const userToken = useCookie('user-token',  { path: '/' }, { expires: 60 * 60 * 24 * 10 })


// vee-validate
const schema = Yup.object().shape({
    firstName: Yup.string().required('First name is required'),
    lastName: Yup.string().required('Last name is required'),
    email: Yup.string()
      .required('Email is required')
      .email('Email is invalid'),
    password: Yup.string()
    .required('Current password is required'),
   
    
});
const authStore = useAuthStore()
const user = ref({
    firstName:'',
    lastName: '',
    email: "",
    password: "",
    newPassword: "",
    confirmPassword:""
})

function getUser(){
    return authStore.getUser
}
watch(getUser, (newVal, oldVal) => {
    if(newVal){
        user.value.firstName = newVal.firstName
        user.value.lastName = newVal.lastName
    }
})

if(authStore.getUser){
    user.value.firstName = authStore.getUser.firstName
    user.value.lastName = authStore.getUser.lastName
    user.value.email = authStore.getUser.email

}
const newPasswordStatus = ref({
    error: false,
    message: ''
})


const confirmStatus = ref({
    error: false,
    message: ''
})


const updated = ref(false)
const serverError = reactive({
    error: false,
    message: ''
})
const {mutate, onDone, onError, loading} = mutation(updateUserMutation, 'user')
function updateUser(){
    confirmStatus.value.error = false;
    newPasswordStatus.value.error = false;

    if(user.value.newPassword && user.value.newPassword.length < 6){
        newPasswordStatus.value.error = true
        newPasswordStatus.value.message = 'Password must be at least 6 characters'
        return
    }

    if(user.value.confirmPassword && !user.value.newPassword){
        newPasswordStatus.value.error = true;
        newPasswordStatus.value.message = 'please provide password'
    }

    if(user.value.newPassword && !user.value.confirmPassword){
        console.log('confirm password is required')
        confirmStatus.value.error = true
        confirmStatus.value.message = 'Please confirm your password'
        return
    }

    if(user.value.newPassword != user.value.confirmPassword){
        confirmStatus.value.error = true
        confirmStatus.value.message = 'Passwords must be the same'
        return
    }
    const data = {
        firstName: user.value.firstName,
        lastName: user.value.lastName,
        email: user.value.email,
        password: user.value.password,
        newPassword: user.value.newPassword,
        
    }
    mutate({data})
}

onDone((result)=>{
    updated.value = true
    userToken.value = 'Bearer '+ result.data.updateUser.token
    authStore.setUserId(result.data.updateUser.id)
    authStore.setToken(userToken.value)
    setTimeout(()=>{
        updated.value = false
    }, 5000)
    console.log('Updated Succesfully')
})

onError(error=>{
    console.log(error)
    serverError.error = true
    serverError.message = error.message
})

definePageMeta({
    layout: "userpanel",
    middleware:["user"]
});
// form handling
// form validation
// form submission
// what are simple requirements

</script>

<template>
    <div v-if="user" class=" lg:ml-6  w-full">
        <Form @change="invalidCredential = false"  @submit="updateUser" :validation-schema="schema" v-slot="{ errors }">
            <!-- first name and last name -->
            <div class=" w-full  mb-4  flex">
                <div  class=" w-full mr-4 text-primary9 ">
                    <label for="firstName" >First Name<span class=" text-red font-bold  text-lg">*</span></label>
                    <Field name="firstName" type="text" id="firstName" v-model="user.firstName" class="w-full border border-gray focus:border-yellow-bright  p-3 " :class="{ 'border-red': errors.firstName }"/>
                    <transition name="error">
                        <span class="text-red text-sm" >{{errors.firstName}}</span>
                    </transition>
                </div>
                <div class=" w-full ml-4 text-primary9">
                    <label for="lastName">Last Name<span class=" text-red font-bold  text-lg">*</span></label>
                    <Field name="lastName" type="text" id="lastName" v-model="user.lastName" class="w-full border border-gray focus:border-yellow-bright  p-3 " :class="{ 'border-red': errors.lastName }"/>
                    <transition name="error">
                        <span class="text-red text-sm" >{{errors.lastName}}</span>
                    </transition>
            
                </div>
            </div> 
             <!--Email  -->
            <div class=" w-full text-primary9">
                <label for="lastName">Email Address<span class=" text-red font-bold  text-lg">*</span></label>
                <Field name="email"  v-model="user.email" type="email" class="w-full p-3 border border-gray     focus:border-yellow-bright "  placeholder="Your email" :class="{ 'border-red': errors.email }" />
                <transition name="error">
                    <span class="text-red text-sm" >{{errors.email}}</span>
                </transition>
            </div>

            <!-- Password Change -->
            <div class=" relative p-4 border border-gray focus:border-yellow-bright  mt-6">
                <p class=" text-primary9 absolute -top-3 z-30 bg-white">Change Password</p>
                <!-- Old password -->
                <div class=" my-6 w-full text-primary9">
                    <label for="password">Current password (leave blank to leave unchanged)<span class=" text-red font-bold  text-lg">*</span></label>
                    <Field name="password"  v-model="user.password" type="password" class="w-full  p-3 border border-gray      focus:border focus:border-yellow-bright focus:border-solid "  placeholder="Your password" :class="{ 'border-red': errors.password }" />
                    <transition name="error">
                        <span class="text-red text-sm" >{{errors.password}}</span>
                    </transition>
                </div>
                <!-- New Password -->
                <div class=" my-6 w-full text-primary9">
                    <label for="newPassword">New password (leave blank to leave unchanged)<span class=" text-red font-bold  text-lg">*</span></label>
                    <Field name="newPassword"  v-model="user.newPassword" type="password" class="w-full  p-3 border border-gray     focus:border focus:border-yellow-bright focus:border-solid "  placeholder="Your password" :class="{ 'border-red': newPasswordStatus.error }" />
                    <transition name="error">
                        <span v-show="newPasswordStatus.error" class="text-red text-sm" >{{newPasswordStatus.message}}</span>
                    </transition>
                </div>
                <!-- Confirm Password -->
                <div class=" mt-6 w-full text-primary9">
                    <label for="confirm">Confirm new password<span class=" text-red font-bold  text-lg">*</span></label>
                    <Field name="confirmPassword" v-model="user.confirmPassword" type="password" class="w-full  p-3 border border-gray       focus:border focus:border-yellow-bright focus:border-solid "  placeholder="Your password" :class="{ 'border-red': confirmStatus.error }" />
                    <transition name="error">
                        <span v-show="confirmStatus.error" class="text-red text-sm" >{{confirmStatus.message}}</span>
                    </transition>
                </div>    
            </div>
            <div class=" relative m-0 bg-gray-dark py-4 w-48 text-center font-bold text-white">
                <button >Save Changes</button>
                <span v-if="loading" a class=" absolute left-20 top-3 animate-spin text-9xl inline-block w-8 h-8 border-[3px] border-current border-t-transparent  text-yellow-bright rounded-full" role="status" aria-label="loading"></span>

            </div>
        </Form>
        <teleport to="body">
            <transition name="book">
                <div v-if="updated"
                    class=" top-28 right-4 text-white font-bold bg-gray-dark p-4 fixed ">
                    <p>Updated Succssefully</p>
                </div>
            </transition>
            <BaseDialog :show="!!serverError.error" :title="serverError.message" @close="serverError.error =false">
                Something Wrong Happened
            </BaseDialog>
        </teleport>  
    </div>
</template>