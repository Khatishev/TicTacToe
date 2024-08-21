<template>
    <div>
        <p>Уровень сложности: {{ difficulty }}</p>
        <button @click="$emit('set-difficulty', 'easy')">Легко</button>
        <button class="btn-medium" @click="$emit('set-difficulty', 'medium')">Средне</button>
        <button @click="$emit('set-difficulty', 'hard')">Сложно</button>
    </div>
</template>

<script>
export default {
    props: {
        difficulty: String
    }
};
</script>