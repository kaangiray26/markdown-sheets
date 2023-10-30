<template>
    <div class="d-flex overflow-x-scroll p-2">
        <table class="table table-bordered mb-0">
            <caption>Created by <a href="https://github.com/kaangiray26" class="text-dark bi bi-github">kaangiray26</a>
            </caption>
            <thead>
                <tr>
                    <th scope="col" class="table-origin table-index"></th>
                    <th v-for="char in alphabet" scope="col" class="table-column" @contextmenu="show_cell_options"
                        @click="edit_cell">
                        <input type="text" class="table-input" :placeholder="char" @keyup.esc="blur" @focusout="focusout"
                            readonly>
                    </th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="index in 8">
                    <th scope="row" class="table-index">{{ index }}</th>
                    <td v-for="_ in alphabet" @contextmenu="show_cell_options" @click="edit_cell">
                        <input type="text" class="table-input" @keyup.esc="blur" @focusout="focusout" readonly>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
    <div v-if="cell_menu_visible" class="cell-menu" :style="{ 'top': cell_menu_top + 'px', 'left': cell_menu_left + 'px' }">
        <div class="d-flex flex-column">
            <ul class="list-group shadow-sm">
                <li class="list-group-item list-group-item-action" @click="copy_cell">Copy</li>
            </ul>
        </div>
    </div>
</template>

<script setup>
import { onMounted, onUnmounted, ref } from 'vue';
const alphabet = 'ABCDEFGH';

const cell_selected = ref(null);

const cell_menu_top = ref(0);
const cell_menu_left = ref(0);
const cell_menu_visible = ref(false);

async function focusout(event) {
    event.target.readOnly = true;
}

async function blur() {
    document.activeElement.readOnly = true;
    document.activeElement.blur();
    cell_menu_visible.value = false;
}

async function edit_cell(event) {
    cell_selected.value = event.target;
    cell_selected.value.readOnly = false;
    cell_selected.value.focus();
}

async function show_cell_options(event) {
    event.preventDefault();
    console.log("Show cell options");
    console.log(event.target);

    cell_menu_top.value = event.clientY;
    cell_menu_left.value = event.clientX;
    cell_menu_visible.value = true;
}

function click_handler(event) {
    if (event.target.closest('.cell-menu') == null) {
        cell_menu_visible.value = false;
    }
}

// Cell menu function
async function copy_cell() {
    if (cell_selected.value == null) {
        cell_menu_visible.value = false;
        return;
    }

    let cell_content = cell_selected.value.value;
    navigator.clipboard.writeText(cell_content)
        .then(() => {
            console.log('Copied to clipboard:', cell_content);
        })

}

onMounted(() => {
    document.addEventListener('click', click_handler);

    // Assign default column names
    let column_names = document.querySelectorAll('.table-column input');
    for (let i = 0; i < column_names.length; i++) {
        column_names[i].value = alphabet[i];
    }
})

onUnmounted(() => {
    document.removeEventListener('click', click_handler);
})
</script>