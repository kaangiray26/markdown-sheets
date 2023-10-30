<template>
    <div class="vwh-100 d-flex flex-column">
        <Topbar />
        <Sheet />
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import Topbar from './Topbar.vue';
import Sheet from './Sheet.vue';
</script>