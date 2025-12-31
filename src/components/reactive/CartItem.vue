<script lang="js" setup>
import { removeCartItem } from "../../utils/store";
import Sample from "../../assets/image-sample.webp";
import { ref, onMounted } from "vue";

const props = defineProps({
    article: Number,
    title: {
        type: String,
        default: "Titre de l'article",
    },
    price: {
        type: Number,
        default: 100,
    },
    image: {
        type: String,
        default: Sample,
    },
    quantity: Number,
    weight: Number,
});

onMounted(() => {
    console.log('📦 CartItem - Poids reçu:', props.weight, 'kg');
    console.log('📦 CartItem - Type du poids:', typeof props.weight);
});

const emit = defineEmits(['update-quantity']);
const localQuantity = ref(props.quantity);

const removeFromCart = () => {
    removeCartItem(Number(props.article));
};

const updateQuantity = () => {
    if (localQuantity.value !== props.quantity) {
        emit('update-quantity', props.article, localQuantity.value);
    }
};
</script>

<template>
    <div class="Article">
        <div class="Article__container">
            <img class="Article__container__image" :src="image" alt="" />
            <div class="Article__container__infos">
                <h3 class="Article__container__infos__title">{{ title }}</h3>
                <span class="Article__container__infos__price"
                    >{{ price }} €</span
                >
                <br />
                <a
                    class="Article__container__infos__remove"
                    @click="removeFromCart()"
                    >Supprimer</a
                >
            </div>
        </div>
        <div class="Article__board">
            <div class="Article__board__quantity">
                <label for="quantity">Quantité</label>
                <br />
                <input 
                    name="quantity" 
                    type="number" 
                    v-model="localQuantity"
                    min="0"
                    @change="updateQuantity"
                />
            </div>
        </div>
    </div>
</template>

<style lang="scss" scoped>
.Article {
    border-bottom: 1px solid rgba(0, 0, 0, 0.1);
    padding: 1.25rem 0;
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 1rem;

    &__container {
        display: flex;
        gap: 1rem;
        &__image {
            width: 96px;
            height: 96px;
            object-fit: cover;
            background: white;
            border-radius: 4px;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
        }
        &__infos {
            display: flex;
            flex-direction: column;
            gap: 0.25rem;
            
            &__title {
                font-family: $font-family-text;
                font-size: 1rem;
                font-weight: 600;
                color: $text-color;
                line-height: 1.4;
            }

            &__price {
                font-family: $font-family-text;
                color: $accent-color;
                font-weight: 700;
            }
            &__remove {
                display: inline-block;
                margin-top: 0.5rem;
                cursor: pointer;
                font-family: $font-family-text;
                color: $accent-color;
                font-weight: 500;
                transition: opacity 0.2s ease;
                &:hover {
                    opacity: 0.7;
                }
            }
        }
    }
    &__board {
        &__quantity {
            width: 120px;
            display: flex;
            flex-direction: column;
            gap: 0.25rem;
            
            label {
                font-family: $font-family-text;
                font-size: 0.9rem;
                color: $text-color;
                opacity: 0.8;
                font-weight: 500;
            }
            input[type="number"] {
                padding: 0.75rem 1rem;
                border: 1px solid rgba(0, 0, 0, 0.15);
                border-radius: 4px;
                font-size: 1rem;
                font-family: $font-family-text;
                background-color: white;
                color: $text-color;
                transition: border-color 0.2s ease;
                
                &:focus {
                    outline: none;
                    border-color: $accent-color;
                }
            }
            &__update {
                margin-top: 0.5rem;
                padding: 0.5rem;
                background-color: $accent-color;
                color: white;
                font-family: $font-family-text;
                border: none;
                border-radius: 4px;
                cursor: pointer;
                font-size: 0.8rem;
                transition: opacity 0.2s ease;

                &:disabled {
                    opacity: 0.5;
                    cursor: not-allowed;
                }

                &:not(:disabled):hover {
                    opacity: 0.85;
                }
            }
        }
    }
}
</style>
