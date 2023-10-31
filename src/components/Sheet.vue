<template>
    <Topbar @download="download" @import_table="import_table" />
    <div class="d-flex overflow-x-scroll p-2">
        <table class="table table-bordered mb-0">
            <caption>Created by <a href="https://github.com/kaangiray26" class="text-dark bi bi-github">kaangiray26</a>
            </caption>
            <thead>
                <tr>
                    <th scope="col" class="table-origin table-index"></th>
                    <th v-for="(header, header_index) in headers" scope="col" class="table-column"
                        @contextmenu="show_column_options" @click="edit_cell">
                        <input type="text" class="table-input" :header_index="header_index" :placeholder="alphabet[index]"
                            :value="header" @keyup.esc="blur" @focusin="edit_cell" @focusout="header_focusout" readonly>
                    </th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(row, row_index) in rows">
                    <th scope="row" class="table-index" @contextmenu="show_row_options">
                        <input type="text" class="table-index-input" :value="row_index + 1" @keyup.esc="blur"
                            @focusout="index_focusout" readonly>
                    </th>
                    <td v-for="(col, col_index) in row.columns" @contextmenu="show_cell_options" @click="edit_cell">
                        <input type="text" class="table-input" :row_index="row_index" :col_index="col_index"
                            :value="col.value" @keyup.esc="blur" @focusin="edit_cell" @focusout="cell_focusout"
                            @keyup.up="go_cell_above" @keyup.down="go_cell_below" readonly>
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
                <li class="list-group-item list-group-item-action d-flex" @click="add_column_before">
                    <span class="bi bi-arrow-90deg-left me-2"></span>
                    <span>Add column before</span>
                </li>
                <li class="list-group-item list-group-item-action d-flex" @click="add_column_after">
                    <span class="bi bi-arrow-90deg-right me-2"></span>
                    <span>Add column after</span>
                </li>
                <li class="list-group-item list-group-item-action d-flex" @click="sort_column_inc">
                    <span class="bi bi-sort-alpha-down me-2"></span>
                    <span>Sort: A-Z</span>
                </li>
                <li class="list-group-item list-group-item-action d-flex" @click="sort_column_dec">
                    <span class="bi bi-sort-alpha-up me-2"></span>
                    <span>Sort: Z-A</span>
                </li>
                <li class="list-group-item list-group-item-action d-flex" @click="delete_column">
                    <span class="bi bi-trash me-2"></span>
                    <span>Delete column</span>
                </li>
            </ul>
        </div>
    </div>
    <div v-if="row_menu_visible" class="cell-menu" :style="{ 'top': row_menu_top + 'px', 'left': row_menu_left + 'px' }">
        <div class="d-flex flex-column">
            <ul class="list-group shadow-sm">
                <li class="list-group-item list-group-item-action d-flex" @click="add_row_above">
                    <span class="bi bi-arrow-90deg-up me-2"></span>
                    <span>Add row above</span>
                </li>
                <li class="list-group-item list-group-item-action d-flex" @click="add_row_belove">
                    <span class="bi bi-arrow-90deg-down me-2"></span>
                    <span>Add row below</span>
                </li>
                <li class="list-group-item list-group-item-action d-flex" @click="delete_row">
                    <span class="bi bi-trash me-2"></span>
                    <span>Delete row</span>
                </li>
            </ul>
        </div>
    </div>
</template>

<script setup>
import { onMounted, onUnmounted, ref } from 'vue';
import Topbar from './Topbar.vue';

const rows = ref([]);
const headers = ref([]);

const num_rows = 8;
const num_columns = 8;
const alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'

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

async function index_focusout(event) {
    return
}

async function header_focusout(event) {
    event.target.readOnly = true;
    let cell_value = event.target.value;
    let header_index = event.target.attributes.header_index.value;

    // Update the header value
    headers.value[header_index] = cell_value;
}

async function cell_focusout(event) {
    event.target.readOnly = true;
    let cell_value = event.target.value;
    let row_index = event.target.attributes.row_index.value;
    let col_index = event.target.attributes.col_index.value;

    // Update the cell value
    rows.value[row_index].columns[col_index].value = cell_value;
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

async function go_cell_above() {
    let row_index = parseInt(cell_selected.value.attributes.row_index.value);
    let col_index = parseInt(cell_selected.value.attributes.col_index.value);

    if (row_index == 0) {
        return
    }

    let next_cell = document.querySelector(`input[row_index="${row_index - 1}"][col_index="${col_index}"]`);
    next_cell.focus();
}

async function go_cell_below() {
    let row_index = parseInt(cell_selected.value.attributes.row_index.value);
    let col_index = parseInt(cell_selected.value.attributes.col_index.value);

    if (row_index == rows.value.length - 1) {
        return
    }

    let next_cell = document.querySelector(`input[row_index="${row_index + 1}"][col_index="${col_index}"]`);
    next_cell.focus();
}

// Table helper functions
function appendRow(index, columns) {
    let arr = [];
    for (let i = 0; i < columns; i++) {
        arr.push({ "value": "" })
    }

    rows.value.push({
        "index": index + 1,
        "columns": arr
    })
}

// Cell menu function
async function copy_cell() {
    if (cell_selected.value == null) {
        hide_all_menus();
        return
    }

    let cell_content = cell_selected.value.value;
    navigator.clipboard.writeText(cell_content)
        .then(() => {
            console.log('Copied to clipboard:', cell_content);
        })

    hide_all_menus();
}

async function add_column_before() {
    // Insert column before column_selected
    let index = parseInt(cell_selected.value.attributes.header_index.value);

    // Insert new column at index
    headers.value.splice(index, 0, alphabet[index]);

    // Update column indexes
    for (let i = index + 1; i < headers.value.length; i++) {
        headers.value[i] = alphabet[i];
    }

    // Insert new column in each row
    for (let i = 0; i < rows.value.length; i++) {
        rows.value[i].columns.splice(index, 0, { "value": "" })
    }

    hide_all_menus();
}

async function add_column_after() {
    // Insert column after column_selected
    let index = parseInt(cell_selected.value.attributes.header_index.value);

    // Insert new column at index
    headers.value.splice(index + 1, 0, alphabet[index + 1]);

    // Update column indexes
    for (let i = index + 2; i < headers.value.length; i++) {
        headers.value[i] = alphabet[i];
    }

    // Insert new column in each row
    for (let i = 0; i < rows.value.length; i++) {
        rows.value[i].columns.splice(index + 1, 0, { "value": "" })
    }

    hide_all_menus();
}

async function add_row_above() {
    // Insert row before row_selected
    let index = parseInt(cell_selected.value.value);

    // Insert new row at index
    let arr = [];
    for (let i = 0; i < headers.value.length; i++) {
        arr.push({ "value": "" })
    }

    rows.value.splice(index - 1, 0, {
        "index": index,
        "columns": arr
    })

    // Update row indexes
    for (let i = index; i < rows.value.length; i++) {
        console
        rows.value[i].index += 1;
    }

    hide_all_menus();
}

async function add_row_belove() {
    // Insert row before row_selected
    let index = parseInt(cell_selected.value.value);

    // Insert new row at index
    let arr = [];
    for (let i = 0; i < headers.value.length; i++) {
        arr.push({ "value": "" })
    }

    rows.value.splice(index, 0, {
        "index": index,
        "columns": arr
    })

    // Update row indexes
    for (let i = index; i < rows.value.length; i++) {
        console
        rows.value[i].index += 1;
    }

    hide_all_menus();
}

async function sort_column_inc() {
    // Sort column in ascending order
    let index = parseInt(cell_selected.value.attributes.header_index.value);

    // Sort column
    rows.value.sort((a, b) => {
        if (a.columns[index].value < b.columns[index].value) {
            return -1;
        }
        if (a.columns[index].value > b.columns[index].value) {
            return 1;
        }
        return 0;
    })

    hide_all_menus();
}

async function sort_column_dec() {
    // Sort column in descending order
    let index = parseInt(cell_selected.value.attributes.header_index.value);

    // Sort column
    rows.value.sort((a, b) => {
        if (a.columns[index].value > b.columns[index].value) {
            return -1;
        }
        if (a.columns[index].value < b.columns[index].value) {
            return 1;
        }
        return 0;
    })

    hide_all_menus();
}

async function delete_column() {
    // Don't delete if there is only one column
    if (headers.value.length == 1) {
        hide_all_menus();
        return
    }

    // Delete column
    let index = parseInt(cell_selected.value.attributes.header_index.value);
    headers.value.splice(index, 1);

    // Update column indexes
    for (let i = index; i < headers.value.length; i++) {
        headers.value[i] = alphabet[i];
    }

    // Delete column in each row
    for (let i = 0; i < rows.value.length; i++) {
        rows.value[i].columns.splice(index, 1)
    }

    hide_all_menus();
}

async function delete_row() {
    // Don't delete if there is only one row
    if (rows.value.length == 1) {
        hide_all_menus();
        return
    }

    // Delete row
    let index = parseInt(cell_selected.value.value);
    rows.value.splice(index - 1, 1);

    // Update row indexes
    for (let i = index - 1; i < rows.value.length; i++) {
        rows.value[i].index -= 1;
    }

    hide_all_menus();
}

// Table imports and exports
async function import_table(file) {
    // Read the file
    let text = await file.text();

    // Clear the table
    rows.value = [];
    headers.value = [];

    // Split the text into lines
    let lines = text.split('\n');

    // Split the first line into headers
    headers.value = lines[0].split('|').filter(val => val.length).map(val => val.trim());

    // Split the rest of the lines into rows
    for (let i = 2; i < lines.length; i++) {
        let row = lines[i].split('|').filter(val => val.length).map(val => val.trim());
        appendRow(i - 2, row.length);
        for (let j = 0; j < row.length; j++) {
            rows.value[i - 2].columns[j].value = row[j];
        }
    }
}

async function download() {
    // Create a new markdown table
    let markdown_table = '';

    // Add headers
    for (let i = 0; i < headers.value.length; i++) {
        markdown_table += '|' + headers.value[i];
    }
    markdown_table += '|\n';

    // Add header separator
    for (let i = 0; i < headers.value.length; i++) {
        markdown_table += '|---';
    }

    markdown_table += '|\n';

    // Add rows
    for (let i = 0; i < rows.value.length; i++) {
        for (let j = 0; j < rows.value[i].columns.length; j++) {
            markdown_table += '|' + rows.value[i].columns[j].value;
        }
        markdown_table += '|\n';
    }

    // Download the markdown table
    let element = document.createElement('a');
    element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(markdown_table));
    element.setAttribute('download', 'table.md');
    element.style.display = 'none';
    document.body.appendChild(element);
    element.click();
}

onMounted(() => {
    document.addEventListener('click', click_handler);

    // Assign default headers
    for (let i = 0; i < num_columns; i++) {
        headers.value.push(alphabet[i]);
    }

    // Initialize the table
    for (let i = 0; i < num_rows; i++) {
        appendRow(i, num_columns)
    }
})

onUnmounted(() => {
    document.removeEventListener('click', click_handler);
})
</script>