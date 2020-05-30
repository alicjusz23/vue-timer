<template>
    <div id="app" class="small-container">
        <h2>Timer</h2>
        <minutnik></minutnik>
    </div>
</template>
<script>
import Minutnik from "@/components/Minutnik.vue"

export default {
    name: "App",
    components: {
        Minutnik
    }

}
</script>
<style>
    h2 {
        text-align: center;
    }

</style>