<template>
    <div class="d-flex flex-column p-2">
        <div class="d-flex">
            <span class="fw-bold text-dark text-nowrap">Markdown Sheets</span>
        </div>
        <div class="d-flex align-items-center">
            <div>
                <div class="dropdown">
                    <button class="btn btn-menu btn-light dropdown-toggle" type="button" data-bs-toggle="dropdown"
                        aria-expanded="false">
                        File
                    </button>
                    <ul class="dropdown-menu">
                        <li>
                            <button class="dropdown-item" @click="import_table">
                                <span class="bi bi-arrow-bar-right me-2"></span>
                                <span>Import table</span>
                            </button>
                        </li>
                        <li>
                            <button class="dropdown-item" @click="download">
                                <span class="bi bi-download me-2"></span>
                                <span>Download</span>
                            </button>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
    <input ref="table_input" type="file" accept="text/markdown" @change="handle_change">
</template>

<script setup>
import { ref } from 'vue';
import { Dropdown } from 'bootstrap';

const table_input = ref(null);
const emit = defineEmits(['import_table', 'download']);

async function import_table() {
    table_input.value.click();
}

async function handle_change() {
    emit('import_table', table_input.value.files[0]);
}

async function download() {
    emit('download');
}
</script>