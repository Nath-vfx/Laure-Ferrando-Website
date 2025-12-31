<template>
    <section class="Contact__form">
        <!-- État de succès -->
        <div v-if="emailStatus === 'success'" class="Contact__form__success">
            <div class="Contact__form__success-icon">✓</div>
            <h2>Message envoyé avec succès !</h2>
            <p>
                Votre message a bien été envoyé. Nous vous répondrons dans les
                plus brefs délais.
            </p>
            <button @click="resetForm" class="Contact__form__success-button">
                Envoyer un nouveau message
            </button>
        </div>

        <!-- État d'erreur -->
        <div v-else-if="emailStatus === 'error'" class="Contact__form__error">
            <div class="Contact__form__error-icon">!</div>
            <h2>Erreur d'envoi</h2>
            <p>{{ errorMessage }}</p>
            <button
                @click="emailStatus = 'idle'"
                class="Contact__form__error-button"
            >
                Réessayer
            </button>
        </div>

        <!-- Formulaire normal -->
        <form v-else ref="form" id="contact-form" @submit.prevent="SendEmail">
            <div class="Contact__form__topic">
                <label for="name">Nom</label>
                <input
                    type="text"
                    name="name"
                    id="name"
                    v-model="fields.name"
                    :disabled="emailStatus === 'sending'"
                />
                <span v-if="errors.name" class="Contact__form__error-text">{{
                    errors.name
                }}</span>
            </div>
            <div class="Contact__form__topic">
                <label for="surname">Prénom</label>
                <input
                    type="text"
                    name="surname"
                    id="surname"
                    v-model="fields.surname"
                    :disabled="emailStatus === 'sending'"
                />
                <span v-if="errors.surname" class="Contact__form__error-text">{{
                    errors.surname
                }}</span>
            </div>
            <div class="Contact__form__topic">
                <label for="email">Adresse mail</label>
                <input
                    type="text"
                    name="email"
                    id="email"
                    v-model="fields.email"
                    :disabled="emailStatus === 'sending'"
                />
                <span v-if="errors.email" class="Contact__form__error-text">{{
                    errors.email
                }}</span>
            </div>
            <div class="Contact__form__topic">
                <label for="subject">Sujet</label>
                <input
                    type="text"
                    name="subject"
                    id="subject"
                    v-model="fields.subject"
                    :disabled="emailStatus === 'sending'"
                />
                <span v-if="errors.subject" class="Contact__form__error-text">{{
                    errors.subject
                }}</span>
            </div>
            <div class="Contact__form__topic">
                <label for="message">Message</label>
                <textarea
                    name="message"
                    id="message"
                    v-model="fields.message"
                    :disabled="emailStatus === 'sending'"
                ></textarea>
                <span v-if="errors.message" class="Contact__form__error-text">{{
                    errors.message
                }}</span>
            </div>
            <button
                class="Contact__form__button"
                type="submit"
                :disabled="emailStatus === 'sending'"
                :class="{
                    'Contact__form__button--loading': emailStatus === 'sending',
                }"
            >
                <span v-if="emailStatus !== 'sending'">Envoyer</span>
                <span v-else class="Contact__form__button-loading">
                    <div class="Contact__form__button-spinner"></div>
                    Envoi en cours...
                </span>
            </button>
        </form>
    </section>
</template>

<script lang="ts" setup>
import emailjs from "@emailjs/browser";
import { ref } from "vue";

const form = ref();
const emailStatus = ref<"idle" | "sending" | "success" | "error">("idle");
const errorMessage = ref("");

const fields = ref({
    name: "",
    surname: "",
    email: "",
    subject: "",
    message: "",
});
const errors = ref({
    name: "",
    surname: "",
    email: "",
    subject: "",
    message: "",
});

function validateEmail(email: string) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

function validate() {
    errors.value = {
        name: fields.value.name ? "" : "Nom requis",
        surname: fields.value.surname ? "" : "Prénom requis",
        email: fields.value.email
            ? validateEmail(fields.value.email)
                ? ""
                : "Email invalide"
            : "Email requis",
        subject: fields.value.subject ? "" : "Sujet requis",
        message: fields.value.message ? "" : "Message requis",
    };
    return Object.values(errors.value).every((e) => !e);
}

const SendEmail = () => {
    if (!validate()) return;

    emailStatus.value = "sending";
    errorMessage.value = "";

    emailjs
        .sendForm(
            import.meta.env.PUBLIC_EMAILJS_SERVICE_ID,
            import.meta.env.PUBLIC_EMAILJS_TEMPLATE_ID,
            form.value,
            {
                publicKey: import.meta.env.PUBLIC_EMAILJS_KEY,
            },
        )
        .then(
            () => {
                emailStatus.value = "success";
            },
            (error) => {
                emailStatus.value = "error";
                errorMessage.value =
                    error?.text ||
                    "Une erreur est survenue lors de l'envoi du message. Veuillez réessayer.";
            },
        );
};

const resetForm = () => {
    // Réinitialiser les champs
    fields.value = {
        name: "",
        surname: "",
        email: "",
        subject: "",
        message: "",
    };

    // Réinitialiser les erreurs
    errors.value = {
        name: "",
        surname: "",
        email: "",
        subject: "",
        message: "",
    };

    // Réinitialiser le statut
    emailStatus.value = "idle";
    errorMessage.value = "";
};
</script>

<style lang="scss" scoped>
.Contact {
    &__form {
        form {
            display: flex;
            flex-direction: column;
            gap: 1.5rem;
        }

        &__topic {
            display: flex;
            flex-direction: column;
            gap: 0.5rem;

            label {
                font-family: $font-family-titles;
                font-weight: 500;
                color: $accent-color;
                font-size: 1rem;
                margin-bottom: 0.25rem;
            }

            input,
            textarea {
                border: 1px solid rgba(0, 0, 0, 0.15);
                font-family: $font-family-text;
                font-size: 1rem;
                padding: 0.75rem 1rem;
                border-radius: 4px;
                transition: border-color 0.2s ease;
                background: white;
                color: $text-color;

                &:focus {
                    outline: none;
                    border-color: $accent-color;
                }

                &::placeholder {
                    color: rgba(0, 0, 0, 0.4);
                }

                &:disabled {
                    background: rgba(0, 0, 0, 0.05);
                    color: rgba(0, 0, 0, 0.5);
                    cursor: not-allowed;
                    opacity: 0.7;
                }
            }

            input {
                height: 3rem;
            }

            textarea {
                height: 10rem;
                resize: vertical;
                min-height: 8rem;
                max-height: 16rem;
            }
        }

        &__button {
            border: none;
            background: $accent-color;
            color: white;
            font-family: $font-family-text;
            font-size: 1rem;
            font-weight: 600;
            padding: 1rem 2.5rem;
            border-radius: 4px;
            cursor: pointer;
            transition: opacity 0.2s ease;
            align-self: flex-start;

            &:hover {
                opacity: 0.85;
            }

            &:active {
                opacity: 0.7;
            }
        }

        // États de succès et d'erreur
        &__success,
        &__error {
            text-align: center;
            padding: 3rem 2rem;

            @media screen and (max-width: 768px) {
                padding: 2rem 1rem;
            }

            h2 {
                font-family: $font-family-titles;
                margin: 1.5rem 0 1rem;
                font-size: 1.75rem;
                color: $text-color;

                @media screen and (max-width: 768px) {
                    font-size: 1.5rem;
                }
            }

            p {
                font-family: $font-family-text;
                color: $text-color;
                opacity: 0.8;
                margin-bottom: 2rem;
                font-size: 1.125rem;
                line-height: 1.6;
            }
        }

        &__success {
            &-icon {
                width: 80px;
                height: 80px;
                margin: 0 auto;
                background: #4caf50;
                color: white;
                border-radius: 50%;
                display: flex;
                align-items: center;
                justify-content: center;
                font-size: 2.5rem;
                font-weight: bold;
            }

            &-button {
                padding: 1rem 2.5rem;
                background: $accent-color;
                color: white;
                border: none;
                border-radius: 4px;
                cursor: pointer;
                transition: opacity 0.2s ease;
                font-weight: 600;
                font-size: 1rem;
                font-family: $font-family-text;

                &:hover {
                    opacity: 0.85;
                }

                &:active {
                    opacity: 0.7;
                }
            }
        }

        &__error {
            &-icon {
                width: 80px;
                height: 80px;
                margin: 0 auto;
                background: #f44336;
                color: white;
                border-radius: 50%;
                display: flex;
                align-items: center;
                justify-content: center;
                font-size: 2.5rem;
                font-weight: bold;
            }

            &-button {
                padding: 1rem 2.5rem;
                background: $accent-color;
                color: white;
                border: none;
                border-radius: 4px;
                cursor: pointer;
                transition: opacity 0.2s ease;
                font-weight: 600;
                font-size: 1rem;
                font-family: $font-family-text;

                &:hover {
                    opacity: 0.85;
                }

                &:active {
                    opacity: 0.7;
                }
            }

            &-text {
                color: #f44336;
                font-size: 0.9rem;
                font-weight: 500;
                margin-top: 0.5rem;
                display: block;
                font-family: $font-family-text;
            }
        }

        &__button {
            &--loading {
                opacity: 0.6;
                cursor: not-allowed;

                &:hover {
                    opacity: 0.6;
                }
            }

            &-loading {
                display: flex;
                align-items: center;
                justify-content: center;
                gap: 0.75rem;
            }

            &-spinner {
                width: 18px;
                height: 18px;
                border: 2px solid rgba(255, 255, 255, 0.3);
                border-top: 2px solid white;
                border-radius: 50%;
                animation: spin 1s linear infinite;
            }
        }
    }
}

// Animations
@keyframes spin {
    0% {
        transform: rotate(0deg);
    }
    100% {
        transform: rotate(360deg);
    }
}

@media screen and (min-width: 768px) {
    .Contact {
        &__form {
            form {
                display: grid;
                grid-template-columns: 1fr 1fr;
                gap: 1.5rem;

                .Contact__form__topic:nth-child(1),
                .Contact__form__topic:nth-child(2) {
                    grid-column: span 1;
                }

                .Contact__form__topic:nth-child(3),
                .Contact__form__topic:nth-child(4),
                .Contact__form__topic:nth-child(5) {
                    grid-column: span 2;
                }
            }

            &__button {
                grid-column: span 2;
                justify-self: start;
            }
        }
    }
}

@media screen and (min-width: 992px) {
    .Contact {
        &__form {
            form {
                display: flex;
                flex-direction: column;
            }

            &__button {
                align-self: flex-start;
            }
        }
    }
}
</style>
