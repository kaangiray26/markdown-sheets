<template>
    <tr>
        <th scope="row" class="table-index" @contextmenu="show_row_options">
            <input type="text" class="table-index-input" :value="index" @keyup.esc="blur" @focusout="focusout" readonly>
        </th>
        <td v-for="_ in alphabet" @contextmenu="show_cell_options" @click="edit_cell">
            <input type="text" class="table-input" @keyup.esc="blur" @focusout="focusout" readonly>
        </td>
    </tr>
</template>

<script setup>
const props = defineProps({
    index: Number,
    alphabet: Array,
    blur: Function,
    focusout: Function,
    edit_cell: Function,
    show_cell_options: Function,
    show_row_options: Function,
})
</script>