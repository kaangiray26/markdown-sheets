<template>
    <div class="d-flex overflow-x-scroll p-2">
        <table class="table table-bordered mb-0">
            <caption>Created by <a href="https://github.com/kaangiray26" class="text-dark bi bi-github">kaangiray26</a>
            </caption>
            <thead>
                <tr>
                    <th scope="col" class="table-origin table-index"></th>
                    <th v-for="char in alphabet" scope="col" class="table-column" @contextmenu="show_column_options"
                        @click="edit_cell">
                        <input type="text" class="table-input" :placeholder="char" @keyup.esc="blur" @focusout="focusout"
                            readonly>
                    </th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="index in 8">
                    <th scope="row" class="table-index" @contextmenu="show_row_options">
                        <input type="text" class="table-index-input" :value="index" @keyup.esc="blur" @focusout="focusout"
                            readonly>
                    </th>
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
    <div v-if="column_menu_visible" class="cell-menu"
        :style="{ 'top': column_menu_top + 'px', 'left': column_menu_left + 'px' }">
        <div class="d-flex flex-column">
            <ul class="list-group shadow-sm">
                <li class="list-group-item list-group-item-action d-flex" @click="copy_cell">
                    <span class="bi bi-copy me-2"></span>
                    <span>Copy</span>
                </li>
                <li class="list-group-item list-group-item-action d-flex" @click="copy_cell">
                    <span class="bi bi-plus me-2"></span>
                    <span>Add column left</span>
                </li>
                <li class="list-group-item list-group-item-action d-flex" @click="copy_cell">
                    <span class="bi bi-plus me-2"></span>
                    <span>Add column right</span>
                </li>
            </ul>
        </div>
    </div>
    <div v-if="row_menu_visible" class="cell-menu" :style="{ 'top': row_menu_top + 'px', 'left': row_menu_left + 'px' }">
        <div class="d-flex flex-column">
            <ul class="list-group shadow-sm">
                <li class="list-group-item list-group-item-action d-flex" @click="add_row_above">
                    <span class="bi bi-plus me-2"></span>
                    <span>Add row above</span>
                </li>
                <li class="list-group-item list-group-item-action d-flex" @click="add_row_belove">
                    <span class="bi bi-plus me-2"></span>
                    <span>Add row below</span>
                </li>
            </ul>
        </div>
    </div>
</template>

<script setup>
import { onMounted, onUnmounted, ref } from 'vue';

const rows = ref([]);
const cols = ref([]);
const alphabet = 'ABCDEFGH';

const cell_selected = ref(null);

const cell_menu_top = ref(0);
const cell_menu_left = ref(0);
const cell_menu_visible = ref(false);

const column_menu_top = ref(0);
const column_menu_left = ref(0);
const column_menu_visible = ref(false);

const row_menu_top = ref(0);
const row_menu_left = ref(0);
const row_menu_visible = ref(false);

async function focusout(event) {
    event.target.readOnly = true;
}

async function blur() {
    document.activeElement.readOnly = true;
    document.activeElement.blur();

    cell_menu_visible.value = false;
    column_menu_visible.value = false;
    row_menu_visible.value = false;
}

async function edit_cell(event) {
    cell_selected.value = event.target;
    cell_selected.value.readOnly = false;
    cell_selected.value.focus();
}

async function show_cell_options(event) {
    event.preventDefault();

    cell_menu_top.value = event.clientY;
    cell_menu_left.value = event.clientX;
    column_menu_visible.value = false;
    row_menu_visible.value = false;
    cell_menu_visible.value = true;
}

async function show_column_options(event) {
    event.preventDefault();

    cell_selected.value = event.target;
    column_menu_top.value = event.clientY;
    column_menu_left.value = event.clientX;
    cell_menu_visible.value = false;
    row_menu_visible.value = false;
    column_menu_visible.value = true;
}

async function show_row_options(event) {
    event.preventDefault();

    cell_selected.value = event.target;
    row_menu_top.value = event.clientY;
    row_menu_left.value = event.clientX;
    cell_menu_visible.value = false;
    column_menu_visible.value = false;
    row_menu_visible.value = true;
}

function click_handler(event) {
    if (event.target.closest('.cell-menu') == null) {
        cell_menu_visible.value = false;
        column_menu_visible.value = false;
        row_menu_visible.value = false;
    }
}

function hide_all_menus() {
    cell_menu_visible.value = false;
    column_menu_visible.value = false;
    row_menu_visible.value = false;
}

// Table helper functions
function appendRow(index, columns) {
    rows.value.push({
        "index": index,
        "columns": Array(columns).fill({ "value": "" })
    })
}

// Cell menu function
async function copy_cell() {
    if (cell_selected.value == null) {
        hide_all_menus()
        return;
    }

    let cell_content = cell_selected.value.value;
    navigator.clipboard.writeText(cell_content)
        .then(() => {
            console.log('Copied to clipboard:', cell_content);
            hide_all_menus();
        })
}

async function add_row_above() {
    // Insert row before row
    let row = cell_selected.value.closest("tr")
    row.before(row.cloneNode(true));
    hide_all_menus();
}

onMounted(() => {
    document.addEventListener('click', click_handler);
    document.addEventListener('keyup', (event) => {
        if (event.key == 'Escape') {
            console.log(event.target);
        }
    });

    // Assign default column names
    let column_names = document.querySelectorAll('.table-column input');
    for (let i = 0; i < column_names.length; i++) {
        column_names[i].value = alphabet[i];
    }

    // Initialize a 8x8 table
    for (let i = 1; i < 9; i++) {
        appendRow(i, 8)
    }
    console.log(rows.value);
})

onUnmounted(() => {
    document.removeEventListener('click', click_handler);
})
</script>