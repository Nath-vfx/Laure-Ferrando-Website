<script setup>
import { ref } from 'vue';
import { addCartItem, cart } from "../../utils/store";

const props = defineProps({
    id: Number,
    name: String,
    image: String,
    price: Number,
    weight: {
        type: [Number, String],
        required: true
    }
});

const showFeedback = ref(false);
const feedbackText = ref('');

const addToCart = () => {
    addCartItem(
        Number(props.id),
        String(props.name),
        String(props.image),
        Number(props.price),
        parseFloat(props.weight)
    );
    feedbackText.value = 'Ajouté au panier ✔';
    showFeedback.value = true;
    setTimeout(() => {
        showFeedback.value = false;
    }, 2000);
};
</script>

<template>
    <div class="AddToCart">
        <button class="Bouton" @click="addToCart()">Ajouter au panier</button>
        <p v-if="showFeedback" class="Feedback">{{ feedbackText }}</p>
    </div>
    
</template>

<style lang="scss" scoped>
.AddToCart {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
}

.Bouton {
    padding: 1rem 2.5rem;
    cursor: pointer;
    border: none;
    font-size: 1rem;
    font-weight: 600;
    font-family: $font-family-text;
    color: white;
    background: $accent-color;
    border-radius: 4px;
    transition: opacity 0.2s ease;
    width: 100%;
    
    &:hover {
        opacity: 0.85;
    }
    
    &:active {
        opacity: 0.7;
    }
}

.Feedback {
    font-size: 0.95rem;
    font-family: $font-family-text;
    color: #198754;
    font-weight: 600;
}
</style>
