<template>
  <div class="relative h-full min-h-screen">
    <div class="px-40">
      <router-view />
    </div>

    <!-- <div class="pt-10"></div> -->
    <!-- <TheCat /> -->
    <!-- <TheFooter /> -->
  </div>
</template>

<script>
import TheNavBar from '@/components/TheNavBar';
import TheFooter from '@/components/gem-farm/TheFooter';
import TheCat from '@/components/gem-farm/TheCat';
export default {
  components: { TheCat, TheFooter, TheNavBar },
};
</script>

<style>
body {
  background-image: url('./assets/background.jpg');
  background-size: cover; /* <------ */
  background-repeat: no-repeat;
  background-position: center center;
}

input[type='radio']:checked + span {
  @apply text-black;
}
</style>
