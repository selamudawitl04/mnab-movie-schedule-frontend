
<script setup>

const showElement = ref(false);

function check(){
    setTimeout(() => {
        showElement.value = true
        
    }, 200);
    // setTimeout(() => {
    //     showElement.value = false
        
    // }, 5000);
}

check()

</script>

<!-- Index Page Header -->
<template>
    <div>
        <section v-if="true" id="header-container" style="" class=" py-60 md:py-80 bg-no-repeat bg-center  bg-cover lg:bg-right  bg-" >
           <div class="">
               <div class="">
                    <transition name="header">    
                        <div class="flex" v-if="showElement" >
                            <div class=" lg:flex-1/2 lg:w-1/2 pl-6 h-auto">
                                <h6 class="  text-yellow-bright text-2xl" >SolFlx</h6>
                                <h2 class="text-white text-6xl font-bold leading-tight"  >Unlimited <span class=" text-yellow-bright">Movie</span>, &amp; More.</h2>
                                <div class="">
                                    <ul class="flex space-x-4 text-white">
                                        <li class="">
                                            <span>Pg 18</span>
                                            <span>hd</span>
                                        </li>
                                        <li class="">
                                            <a href="#">Romance,</a>
                                            <a href="#">Drama</a>
                                        </li>
                                        <li class="">
                                            <span><i class=""></i> 2021</span>
                                            <span><i class=""></i> 128 min</span>
                                        </li>
                                    </ul>
                                </div>
                                <div  class="py-10">    
                                        <router-link class="text-white  bg-primary2 font-bold rounded-full border-4 
                                            border-solid border-yellow-bright py-4  px-8 text-center hover:bg-yellow-bright hover:text-gray-dark  " 
                                            to="/auth/signup">Register Now
                                        </router-link>
                                    </div>
                            </div>
                        </div>
                    </transition>
               </div>
           </div>
       </section>
    </div>  
</template>

<style scoped>
#header-container{
    background-image: url(../../assets/img/header-bg.jpg);
}
.header-enter-from{
  opacity: 0;
  transform: translateY(40px);
}
.header-enter-to{
  opacity: 1;
  transform: translateX(0);
}
.header-enter-active{
  transition: all 2s linear;
}

</style>