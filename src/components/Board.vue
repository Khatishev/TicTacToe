<template>
    <div class="board">
        <div v-for="(cell, index) in board" :key="index" :ref="`cell-${index}`" class="cell" @click="$emit('make-move', index)">
            {{ cell }}
        </div>
    </div>
</template>

<script>
export default {
    props: {
        board: Array
    }
};
</script>