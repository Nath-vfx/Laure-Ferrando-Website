<script lang="js" setup>
import CartItem from "./CartItem.vue";
import { useStore } from "@nanostores/vue";
import { cart, updateCartItemQuantity } from "../../utils/store";
import { computed, onMounted, ref, watch } from "vue";

const isClient = ref(false);
const subTotal = ref(0);
const shipping = ref(0);
const totalWeight = ref(0);
const total = computed(() => subTotal.value + shipping.value);
const showShippingInfo = ref(false);
const isLoading = ref(true);

const $cart = ref(useStore(cart));

const handleQuantityUpdate = async (articleId, newQuantity) => {
    try {
        isLoading.value = true;
        updateCartItemQuantity(articleId, newQuantity);
    } catch (error) {
        console.error('Erreur lors de la mise à jour de la quantité:', error);
    } finally {
        isLoading.value = false;
    }
};

const calculateShipping = (weight) => {
    if (weight < 1) {
        return 5;
    } else if (weight >= 1 && weight <= 5) {
        return 10;
    } else {
        return 15;
    }
};

const cost = () => {
    if ($cart.value.length > 0) {
        let e = 0;
        let totalW = 0;
        $cart.value.forEach((item) => {
            e += item.quantity * item.price;
            totalW += item.quantity * (item.weight || 0);
        });
        subTotal.value = e;
        totalWeight.value = parseFloat(totalW.toFixed(2));
        shipping.value = calculateShipping(totalWeight.value);
    }
};

watch(
    $cart,
    (newCart) => {
        if (newCart && newCart.length > 0) {
            console.log("Panier mis à jour :", newCart);
            newCart.forEach(item => {
                console.log(`📦 Cart - Article "${item.name}" - Poids:`, item.weight, 'kg');
                console.log(`📦 Cart - Type du poids:`, typeof item.weight);
            });
            cost();
            isLoading.value = false;
        } else {
            isLoading.value = false;
        }
    },
    { immediate: true, deep: true },
);

onMounted(() => {
    isClient.value = true;
    if ($cart.value.length === 0) {
        isLoading.value = false;
    } else {
        cost();
        isLoading.value = false;
    }
});
</script>

<template>
    <div class="cart">
        <div v-if="isClient" class="cart__wrapper">
            <template v-if="$cart && $cart.length > 0">
                <div class="cart__header">
                    <h1>Mon panier</h1>
                </div>
                
                <div class="cart__main">
                    <div class="cart__items">
                        <ul>
                            <li v-for="article in $cart" :key="article.id">
                        <CartItem
                            :article="article.id"
                            :title="article.name"
                            :image="article.image"
                            :price="article.price"
                            :quantity="article.quantity"
                                    :weight="parseFloat(article.weight)"
                                    @update-quantity="handleQuantityUpdate"
                        />
                    </li>
                </ul>
            </div>

                    <div class="cart__summary">
                        <h2>Résumé de la commande</h2>
                        <table>
                    <tbody>
                                <tr>
                            <td>Sous-total</td>
                            <td>{{ subTotal }} €</td>
                        </tr>
                                <tr>
                                    <td>Poids total</td>
                                    <td>{{ totalWeight }} kg</td>
                                </tr>
                                <tr>
                                    <td>
                                        Livraison
                                        <button 
                                            class="cart__summary__info-button"
                                            @click="showShippingInfo = true"
                                            aria-label="Informations sur les frais de port"
                                        >
                                            <svg 
                                                xmlns="http://www.w3.org/2000/svg" 
                                                width="16" 
                                                height="16" 
                                                viewBox="0 0 24 24" 
                                                fill="none" 
                                                stroke="currentColor" 
                                                stroke-width="2" 
                                                stroke-linecap="round" 
                                                stroke-linejoin="round"
                                            >
                                                <circle cx="12" cy="12" r="10"></circle>
                                                <line x1="12" y1="16" x2="12" y2="12"></line>
                                                <line x1="12" y1="8" x2="12.01" y2="8"></line>
                                            </svg>
                                        </button>
                                    </td>
                            <td>{{ shipping }} €</td>
                        </tr>
                                <tr class="total">
                            <td>Total</td>
                            <td>{{ total }} €</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
                <div class="cart__actions">
                    <a href="/paiement" class="cart__actions__button">
                        Procéder au paiement
                    </a>
                </div>
            </template>

            <div v-else class="cart__empty">
                <div class="cart__empty__content">
                    <svg width="80" height="80" viewBox="0 0 24 24" fill="none" class="cart__empty__icon">
                        <path d="M6 2L3 6V20C3 20.5304 3.21071 21.0391 3.58579 21.4142C3.96086 21.7893 4.46957 22 5 22H19C19.5304 22 20.0391 21.7893 20.4142 21.4142C20.7893 21.0391 21 20.5304 21 20V6L18 2H6Z" stroke="#b39966" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                        <path d="M3 6H21" stroke="#b39966" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                        <path d="M16 10C16 11.0609 15.5786 12.0783 14.8284 12.8284C14.0783 13.5786 13.0609 14 12 14C10.9391 14 9.92172 13.5786 9.17157 12.8284C8.42143 12.0783 8 11.0609 8 10" stroke="#b39966" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                    </svg>
                    <h2 class="cart__empty__title">Votre panier est vide</h2>
                    <p class="cart__empty__message">Il semble que vous n'avez pas encore ajouté d'articles à votre panier.</p>
                    <a href="/boutique" class="cart__empty__button">
                        Continuer mes achats
                    </a>
                </div>
            </div>
        </div>
        <div v-else class="cart__wrapper">
            <div class="cart__loading">
                <div class="cart__loader"></div>
                <p>Chargement de votre panier...</p>
            </div>
        </div>

        <!-- Modal des frais de port -->
        <div v-if="showShippingInfo" class="cart__modal">
            <div class="cart__modal__overlay" @click="showShippingInfo = false"></div>
            <div class="cart__modal__content">
                <button 
                    class="cart__modal__close"
                    @click="showShippingInfo = false"
                    aria-label="Fermer"
                >
                    ×
                </button>
                <h2>Calcul des frais de port</h2>
                <div class="cart__modal__shipping-info">
                    <p>Les frais de port sont calculés en fonction du poids total de votre commande :</p>
                    <ul>
                        <li>
                            <span class="cart__modal__shipping-info__weight">Moins de 1 kg</span>
                            <span class="cart__modal__shipping-info__price">5 €</span>
                        </li>
                        <li>
                            <span class="cart__modal__shipping-info__weight">De 1 à 5 kg</span>
                            <span class="cart__modal__shipping-info__price">10 €</span>
                        </li>
                        <li>
                            <span class="cart__modal__shipping-info__weight">Plus de 5 kg</span>
                            <span class="cart__modal__shipping-info__price">15 €</span>
                        </li>
                    </ul>
                    <p class="cart__modal__shipping-info__current">
                        Votre commande pèse actuellement <strong>{{ totalWeight }} kg</strong>
                    </p>
                </div>
            </div>
        </div>
    </div>
</template>

<style lang="scss" scoped>
$cart-side-width: 460px;
.cart {
    margin-top: 4rem;
    min-height: 50vh;
    
    &__wrapper {
        width: 100%;
        max-width: 1280px;
        margin: 0 auto;
        padding: 0 2rem;
    }

    &__header {
        margin-bottom: 3rem;
        
        h1 {
            font-family: $font-family-titles;
            font-size: 2.5rem;
            color: $text-color;
            text-align: center;
            
            @media (max-width: 768px) {
                font-size: 2rem;
            }
        }
    }

    &__main {
        display: grid;
        grid-template-columns: 1fr;
        gap: 2rem;

        @media screen and (min-width: 992px) {
            grid-template-columns: 2fr 1fr;
            }
    }

    &__items {
        ul {
            list-style: none;
            padding: 0;
            margin: 0;
        }
    }

    &__summary {
        background: white;
        padding: 2rem;
        border-radius: 4px;
        height: fit-content;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
        width: $cart-side-width;
        max-width: $cart-side-width;
        margin-left: auto;

        @media screen and (max-width: 992px) {
            width: 100%;
            max-width: 100%;
            margin-left: 0;
        }

        h2 {
            margin-bottom: 1.5rem;
            font-family: $font-family-titles;
            font-size: 1.5rem;
            color: $accent-color;
        }

        table {
                width: 100%;
                margin: 1rem 0;
                border-collapse: collapse;

                tr {
                    height: 56px;
                    border-bottom: 1px solid rgba(0, 0, 0, 0.1);
                
                    td {
                        padding: 0 1.25rem;
                        font-family: $font-family-text;
                        font-weight: 500;
                        color: $text-color;
                        
                        &:last-child {
                            text-align: right;
                            font-weight: 600;
                        }
                }

                &.total {
                    background: $accent-color;
                    border-bottom: none;
                    
                    td {
                        color: white;
                        font-weight: 700;
                        font-size: 1.1rem;
                    }
                }
            }
        }

        &__info-button {
            background: none;
            border: none;
            cursor: pointer;
            padding: 0.25rem;
            vertical-align: middle;
            transition: opacity 0.2s ease;
            color: $text-color;
            opacity: 0.6;
            display: inline-flex;
            align-items: center;
            justify-content: center;
            border-radius: 50%;
            width: 24px;
            height: 24px;

            &:hover {
                opacity: 1;
            }

            svg {
                width: 16px;
                height: 16px;
            }
        }
    }

    &__empty {
        padding: 4rem 2rem;
        text-align: center;
        margin: 2rem auto;
        max-width: 600px;

        &__content {
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        &__icon {
            margin-bottom: 1.5rem;
            color: $accent-color;
        }

        &__title {
            font-family: $font-family-titles;
            font-size: 1.75rem;
            color: $text-color;
            margin-bottom: 0.75rem;
        }

        &__message {
            font-family: $font-family-text;
            color: $text-color;
            opacity: 0.8;
            margin-bottom: 2rem;
            line-height: 1.6;
            max-width: 80%;
        }

        &__button {
            background: $accent-color;
            color: white;
            font-family: $font-family-text;
            border: none;
            padding: 1rem 2.5rem;
            border-radius: 4px;
            text-decoration: none;
            font-weight: 600;
            display: inline-block;
            transition: opacity 0.2s ease;
            font-size: 1rem;
            cursor: pointer;

            &:hover {
                opacity: 0.85;
            }
            
            &:active {
                opacity: 0.7;
            }
        }
    }

    &__modal {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        display: flex;
        align-items: center;
        justify-content: center;
        z-index: 1000;

        &__overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.5);
        }

        &__content {
            position: relative;
            background: white;
            padding: 2.5rem;
            border-radius: 4px;
            max-width: 90%;
            width: $cart-side-width;
            max-height: 90vh;
            overflow-y: auto;
            z-index: 1;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);

            h2 {
                margin-bottom: 1.5rem;
                font-family: $font-family-titles;
                font-size: 1.5rem;
                color: $accent-color;
            }
        }

        &__close {
            position: absolute;
            top: 1rem;
            right: 1rem;
            background: none;
            border: none;
            font-size: 1.5rem;
            cursor: pointer;
            padding: 0.5rem;
            line-height: 1;
            transition: opacity 0.3s ease;

            &:hover {
                opacity: 0.7;
            }
        }

        &__shipping-info {
            font-family: $font-family-text;
            
            ul {
                list-style: none;
                padding: 0;
                margin: 1.5rem 0;

                li {
                    display: flex;
                    justify-content: space-between;
                    padding: 0.75rem 0;
                    border-bottom: 1px solid rgba(0, 0, 0, 0.1);

                    &:last-child {
                        border-bottom: none;
                    }
                }
            }

            &__weight {
                font-weight: 500;
                color: $text-color;
            }

            &__price {
                color: $accent-color;
                font-weight: 600;
            }

            &__current {
                margin-top: 1.5rem;
                padding-top: 1.5rem;
                border-top: 1px solid rgba(0, 0, 0, 0.1);
                text-align: center;
                color: $text-color;
                opacity: 0.8;

                strong {
                    color: $text-color;
                    opacity: 1;
                }
            }
        }
    }

    &__actions {
        margin-top: 2rem;
        text-align: right;
        width: 100%;
        
        width: $cart-side-width;
        max-width: $cart-side-width;
        margin-left: auto;
        
        @media screen and (max-width: 992px) {
            width: 100%;
            max-width: 100%;
        }

        &__button {
            display: inline-block;
            padding: 1rem 2.5rem;
            background: $accent-color;
            color: white;
            font-family: $font-family-text;
            text-decoration: none;
            font-size: 1rem;
            font-weight: 600;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            transition: opacity 0.2s ease;
            width: 100%;
            text-align: center;

            &:hover {
                opacity: 0.85;
            }
            
            &:active {
                opacity: 0.7;
            }
        }
    }
}

.cart__loading {
    text-align: center;
    padding: 5rem 2rem;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 2rem;

    p {
        font-family: $font-family-text;
        font-size: 1.125rem;
        margin: 0;
        color: $text-color;
        opacity: 0.8;
    }
}

.cart__loader {
    width: 60px;
    height: 60px;
    border: 4px solid rgba(0, 0, 0, 0.1);
    border-top: 4px solid $accent-color;
    border-radius: 50%;
    animation: spin 1s linear infinite;
}

@keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
}
</style>
