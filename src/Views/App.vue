<template>
    <main id="app">
        <div id="model-container"></div>
        <!-- TopHead is the header with the information about the app -->
        <section class="container chat-container">
            <!-- Error component is for displaying errors -->
            <Error v-if="error" :error="error" />

            <!-- Welcome component is for onboarding experience and language picker -->
            <!-- <Welcome v-if="app && messages.length == 0" :app="app" /> -->



            <!-- Messages Table -->
            <section v-else aria-live="polite">
                <div v-for="message in messages" id="message" :key="message.responseId">
                    <!-- My message -->
                    <BubbleWrapper>
                        <Bubble v-if="message.queryResult.queryText" :text="message.queryResult.queryText" me />
                    </BubbleWrapper>

                    <!-- Dialogflow Components -->
                    <RichComponent v-for="(component, component_id) in message.queryResult.fulfillmentMessages"
                        :key="component_id">
                        <!-- Text (https://cloud.google.com/dialogflow/docs/reference/rest/v2beta1/projects.agent.intents#Text) -->
                        <Bubble v-if="component.text" :text="component.text.text[0]" />

                        <!-- SimpleResponses (https://cloud.google.com/dialogflow/docs/reference/rest/v2beta1/projects.agent.intents#SimpleResponses) -->
                        <Bubble v-if="component.simpleResponses"
                            :text="component.simpleResponses.simpleResponses[0].displayText || component.simpleResponses.simpleResponses[0].textToSpeech" />

                        <!-- RbmText (https://cloud.google.com/dialogflow/docs/reference/rest/v2beta1/projects.agent.intents#rbmtext) -->
                        <div v-if="component.rbmText">
                            <Bubble :text="component.rbmText.text" />
                            <div v-for="(suggestion, suggestion_id) in component.rbmText.rbmSuggestion"
                                :key="suggestion_id">
                                <CardButton v-if="suggestion.reply" :title="suggestion.reply.text"
                                    @click.native="send({ text: suggestion.reply.text.postbackData })" />

                                <CardButton v-if="suggestion.action" :title="suggestion.action.text"
                                    :uri="suggestion.action.openUrl.uri" />
                            </div>
                        </div>

                        <!-- Card (https://cloud.google.com/dialogflow/docs/reference/rest/v2beta1/projects.agent.intents#Card) -->
                        <Card v-if="component.card" :title="component.card.title" :subtitle="component.card.subtitle"
                            :image-uri="component.card.imageUri">
                            <CardButton v-for="(button, button_id) in component.card.buttons" :key="button_id"
                                :uri="button.postback" :title="button.text" />
                        </Card>

                        <!-- BasicCard (https://cloud.google.com/dialogflow/docs/reference/rest/v2beta1/projects.agent.intents#BasicCard) -->
                        <Card v-if="component.basicCard" :title="component.basicCard.title"
                            :subtitle="component.basicCard.subtitle" :image-uri="component.basicCard.image.imageUri"
                            :image-title="component.basicCard.image.accessibilityText"
                            :text="component.basicCard.formattedText">
                            <CardButton v-for="(button, button_id) in component.basicCard.buttons" :key="button_id"
                                :uri="button.openUriAction.uri" :title="button.title" />
                        </Card>

                        <!-- RbmStandaloneCard (https://cloud.google.com/dialogflow/docs/reference/rest/v2beta1/projects.agent.intents#rbmstandalonecard) -->
                        <Card v-if="component.rbmStandaloneRichCard"
                            :title="component.rbmStandaloneRichCard.cardContent.title"
                            :image-uri="component.rbmStandaloneRichCard.cardContent.media.fileUri"
                            :text="component.rbmStandaloneRichCard.cardContent.description">
                            <div v-for="(suggestion, suggestion_id) in component.rbmStandaloneRichCard.cardContent.suggestions"
                                :key="suggestion_id">
                                <CardButton v-if="suggestion.reply" :title="suggestion.reply.text"
                                    @click.native="send({ text: suggestion.reply.text.postbackData })" />
                                <CardButton v-if="suggestion.action" :title="suggestion.action.text"
                                    :uri="suggestion.action.openUrl.uri" />
                            </div>
                        </Card>

                        <!-- CarouselSelect (https://cloud.google.com/dialogflow/docs/reference/rest/v2beta1/projects.agent.intents#CarouselSelect) -->
                        <Carousel v-if="component.carouselSelect">
                            <Card v-for="item in component.carouselSelect.items" :key="item.info.key" :title="item.title"
                                :image-uri="item.image.imageUri" :image-title="item.image.accessibilityText"
                                :text="item.description" @click.native="send({ text: item.info.key })" />
                        </Carousel>

                        <!-- RbmCarouselCard (https://cloud.google.com/dialogflow/docs/reference/rest/v2beta1/projects.agent.intents#rbmcarouselcard) -->
                        <Carousel v-if="component.rbmCarouselRichCard">
                            <Card v-for="(card, card_id) in component.rbmCarouselRichCard.cardContents" :key="card_id"
                                :title="card.title" :image-uri="card.media.fileUri" :text="card.description">
                                <div v-for="(suggestion, suggestion_id) in card.suggestions" :key="suggestion_id">
                                    <CardButton v-if="suggestion.reply" :title="suggestion.reply.text"
                                        @click.native="send({ text: suggestion.reply.text.postbackData })" />
                                    <CardButton v-if="suggestion.action" :title="suggestion.action.text"
                                        :uri="suggestion.action.openUrl.uri" />
                                </div>
                            </Card>
                        </Carousel>

                        <!-- ListSelect (https://cloud.google.com/dialogflow/docs/reference/rest/v2beta1/projects.agent.intents#ListSelect) -->
                        <List v-if="component.listSelect" :title="component.listSelect.title"
                            :subtitle="component.listSelect.subtitle">
                            <ListItem v-for="item in component.listSelect.items" :key="item.info.key" :title="item.title"
                                :description="item.description" :image-uri="item.image.imageUri"
                                :image-title="item.image.accessibilityText" @click.native="send({ text: item.info.key })" />
                        </List>

                        <!-- Image (https://cloud.google.com/dialogflow/docs/reference/rest/v2beta1/projects.agent.intents#Image) -->
                        <Picture v-if="component.image" :uri="component.image.imageUri"
                            :title="component.image.accessibilityText" />

                        <!-- Media (https://cloud.google.com/dialogflow/docs/reference/rest/v2beta1/projects.agent.intents#MediaContent) -->
                        <div v-if="component.mediaContent && component.mediaContent.mediaObjects">
                            <Media v-for="(media, media_id) in component.mediaContent.mediaObjects" :key="media_id"
                                :name="media.name" :description="media.description"
                                :icon-uri="media.icon ? media.icon.imageUri : media.largeImage.imageUri"
                                :icon-title="media.icon ? media.icon.accessibilityText : media.largeImage.accessibilityText"
                                :uri="media.contentUrl" />
                        </div>

                        <!-- TableCard (https://cloud.google.com/dialogflow/docs/reference/rest/v2beta1/projects.agent.intents#tablecard) -->
                        <TableCard v-if="component.tableCard" :title="component.tableCard.title"
                            :subtitle="component.tableCard.subtitle" :image-uri="component.tableCard.image.imageUri"
                            :image-title="component.tableCard.image.accessibilityText"
                            :header="component.tableCard.columnProperties" :rows="component.tableCard.rows">
                            <CardButton v-for="(button, button_id) in component.tableCard.buttons" :key="button_id"
                                :uri="button.openUriAction.uri" :title="button.title" />
                        </TableCard>
                    </RichComponent>

                    <!-- Actions on Google Components -->
                    <section v-if="message.queryResult.webhookPayload && message.queryResult.webhookPayload.google">
                        <RichComponent
                            v-for="(component, component_id) in message.queryResult.webhookPayload.google.richResponse.items"
                            :key="component_id">
                            <!-- Simple response (https://developers.google.com/actions/assistant/responses#simple_response) -->
                            <Bubble v-if="component.simpleResponse"
                                :text="component.simpleResponse.displayText || component.simpleResponse.textToSpeech" />

                            <!-- Basic card (https://developers.google.com/actions/assistant/responses#basic_card) -->
                            <Card v-if="component.basicCard" :title="component.basicCard.title"
                                :subtitle="component.basicCard.subtitle" :image-uri="component.basicCard.image.url"
                                :image-title="component.basicCard.image.accessibilityText"
                                :text="component.basicCard.formattedText">
                                <CardButton v-for="(button, button_id) in component.basicCard.buttons" :key="button_id"
                                    :uri="button.openUrlAction.url" :title="button.title" />
                            </Card>

                            <!-- Browsing Carousel (https://developers.google.com/actions/assistant/responses#browsing_carousel) -->
                            <List v-if="component.carouselBrowse">
                                <ListItem v-for="(item, item_id) in component.carouselBrowse.items" :key="item_id"
                                    :uri="item.openUrlAction.url" :title="item.title" :description="item.description"
                                    :footer="item.footer" :image-uri="item.image.url"
                                    :image-title="item.image.accessibilityText" />
                            </List>

                            <!-- Media responses (https://developers.google.com/actions/assistant/responses#media_responses) -->
                            <div v-if="component.mediaResponse && component.mediaResponse.mediaObjects">
                                <Media v-for="(media, media_id) in component.mediaResponse.mediaObjects" :key="media_id"
                                    :name="media.name" :description="media.description" :icon-uri="media.icon.url"
                                    :icon-title="media.icon.accessibilityText" :uri="media.contentUrl" />
                            </div>

                            <!-- Table cards (https://developers.google.com/actions/assistant/responses#table_cards) -->
                            <TableCard v-if="component.tableCard" :title="component.tableCard.title"
                                :subtitle="component.tableCard.subtitle" :image-uri="component.tableCard.image.url"
                                :image-title="component.tableCard.image.accessibilityText"
                                :header="component.tableCard.columnProperties" :rows="component.tableCard.rows">
                                <CardButton v-for="(button, button_id) in component.tableCard.buttons" :key="button_id"
                                    :uri="button.openUrlAction.url" :title="button.title" />
                            </TableCard>
                        </RichComponent>

                        <!-- Visual Selection Responses (https://developers.google.com/actions/assistant/responses#visual_selection_responses) -->
                        <RichComponent
                            v-for="(component, component_id) in message.queryResult.webhookPayload.google.systemIntent"
                            :key="component_id">
                            <!-- List (https://developers.google.com/actions/assistant/responses#list) -->
                            <List v-if="component.listSelect" :title="component.listSelect.title"
                                :subtitle="component.listSelect.subtitle">
                                <ListItem v-for="item in component.listSelect.items" :key="item.optionInfo.key"
                                    :title="item.title" :description="item.description" :image-uri="item.image.url"
                                    :image-title="item.image.accessibilityText"
                                    @click.native="send({ text: item.optionInfo.key })" />
                            </List>

                            <!-- Carousel (https://developers.google.com/actions/assistant/responses#carousel) -->
                            <Carousel v-if="component.carouselSelect">
                                <Card v-for="item in component.carouselSelect.items" :key="item.optionInfo.key"
                                    :title="item.title" :image-uri="item.image.url"
                                    :image-title="item.image.accessibilityText" :text="item.description"
                                    @click.native="send({ text: item.optionInfo.key })" />
                            </Carousel>
                        </RichComponent>
                    </section>
                </div>
                <div v-if="loading" id="message">
                    <!-- My message (Loading) -->
                    <BubbleWrapper>
                        <Bubble me loading aria-hidden="true" />
                    </BubbleWrapper>

                    <!-- Default / Webhook bubble (Loading) -->
                    <Bubble loading aria-hidden="true" />
                </div>
            </section>
        </section>

        <!-- ChatInput is made for submitting queries and displaying suggestions -->
        <ChatInput ref="input" @submit="send">
            <!-- Suggestion chips
                https://developers.google.com/actions/assistant/responses#suggestion_chips
                https://cloud.google.com/dialogflow/docs/reference/rest/v2beta1/projects.agent.intents#QuickReplies
                https://cloud.google.com/dialogflow/docs/reference/rest/v2beta1/projects.agent.intents#Suggestions
            -->
            <Suggestion v-for="(suggestion, suggestion_id) in suggestions.text_suggestions" :key="suggestion_id"
                :title="suggestion" @click.native="send({ text: suggestion })" />

            <!-- Link suggestion chips
                https://developers.google.com/actions/assistant/responses#suggestion_chips
                https://cloud.google.com/dialogflow/docs/reference/rest/v2beta1/projects.agent.intents#LinkOutSuggestion
            -->
            <Suggestion v-if="suggestions.link_suggestion" :title="suggestions.link_suggestion.destinationName"
                :uri="suggestions.link_suggestion.uri || suggestions.link_suggestion.url" />
        </ChatInput>
    </main>
</template>

<style lang="sass">
@import '@/Style/Theme.sass'

body
    margin: 0
    padding: 0
    font-family: var(--font)
    font-display: swap
    background-color: var(--background)

.container
    max-width: 500px
    margin-left: auto
    margin-right: auto
    padding: 12px
    position: relative

.chat-container
    margin-top: 500px

#model-container
    position: fixed
    top: 0
    left: 50%
    transform: translate(-50%, 0)
    z-index: 2
</style>

<style lang="sass" scoped>
.chat-container
    padding-top: 80px
    padding-bottom: 125px
</style>

<script>
import Welcome from './Welcome.vue'

import Error from '@/Components/Parts/Error.vue'
import TopHead from '@/Components/Parts/TopHead.vue'
import ChatInput from '@/Components/Parts/ChatInput.vue'

import RichComponent from '@/Components/Rich/Component.vue'
import Bubble from '@/Components/Rich/Bubble.vue'
import BubbleWrapper from '@/Components/Rich/BubbleWrapper.vue'
import Card from '@/Components/Rich/Card.vue'
import CardButton from '@/Components/Rich/CardButton.vue'
import Carousel from '@/Components/Rich/Carousel.vue'
import List from '@/Components/Rich/List.vue'
import ListItem from '@/Components/Rich/ListItem.vue'
import Picture from '@/Components/Rich/Picture.vue'
import Media from '@/Components/Rich/Media.vue'
import TableCard from '@/Components/Rich/TableCard.vue'
import Suggestion from '@/Components/Rich/Suggestion.vue'
import { Buffer } from 'buffer';
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';

import { v1 as uuidv1 } from 'uuid';
import Vue from 'vue'
import VueResource from 'vue-resource'
Vue.use(VueResource);

const defaultFallbackPhrases = [
    'I didn\'t get that. Can you say it again?',
    'I missed what you said. What was that?',
    'Sorry, could you say that again?',
    'Sorry, can you say that again?',
    'Can you say that again?',
    'Sorry, I didn\'t get that. Can you rephrase?',
    'Sorry, what was that?',
    'One more time?',
    'What was that?',
    'Say that one more time?',
    'I didn\'t get that. Can you repeat?',
    'I missed that, say that again?',
];

export default {
    name: 'App',
    components: {
        Welcome,
        Error,
        TopHead,
        ChatInput,
        RichComponent,
        Bubble,
        BubbleWrapper,
        Card,
        CardButton,
        Carousel,
        List,
        ListItem,
        Picture,
        Media,
        TableCard,
        Suggestion
    },
    mounted() {
        this.loadBaseModel();
    },
    data() {
        return {
            app: null,
            messages: [],
            language: '',
            session: '',
            muted: this.config.muted,
            loading: false,
            error: null,
            // client: new Client(this.config.endpoint).connect()
        }
    },
    computed: {
        /* The code below is used to extract suggestions from last message, to display it on ChatInput */
        suggestions() {
            if (this.messages.length > 0) {
                const last_message = this.messages[this.messages.length - 1]
                const suggestions = []

                /* Dialogflow Suggestions */
                for (const component in last_message.queryResult.fulfillmentMessages) {
                    if (last_message.queryResult.fulfillmentMessages[component].suggestions) suggestions.text_suggestions = last_message.queryResult.fulfillmentMessages[component].suggestions.suggestions.map(suggestion => suggestion.title)
                    if (last_message.queryResult.fulfillmentMessages[component].linkOutSuggestion) suggestions.link_suggestion = last_message.queryResult.fulfillmentMessages[component].linkOutSuggestion
                    if (last_message.queryResult.fulfillmentMessages[component].quickReplies) suggestions.text_suggestions = last_message.queryResult.fulfillmentMessages[component].quickReplies.quickReplies
                }

                /* Google Suggestions */
                if (last_message.queryResult.webhookPayload && last_message.queryResult.webhookPayload.google) {
                    for (const component in last_message.queryResult.webhookPayload.google) {
                        if (last_message.queryResult.webhookPayload.google[component].suggestions) suggestions.text_suggestions = last_message.queryResult.webhookPayload.google[component].suggestions.map(suggestion => suggestion.title)
                        if (last_message.queryResult.webhookPayload.google[component].linkOutSuggestion) suggestions.link_suggestion = last_message.queryResult.webhookPayload.google[component].linkOutSuggestion
                    }
                }

                return suggestions
            }

            return {
                text_suggestions: this.config.start_suggestions // <- if no messages are present, return start_suggestions, from config.js to help user figure out what he can do with your application
            }
        }
    },
    watch: {
        /* This function is triggered, when new messages arrive */
        messages(messages) {
            if (this.history()) sessionStorage.setItem('message_history', JSON.stringify(messages)) // <- Save history if the feature is enabled
        },
        /* This function is triggered, when request is started or finished */
        loading() {
            setTimeout(() => {
                const app = document.querySelector('#app') // <- We need to scroll down #app, to prevent the whole page jumping to bottom, when using in iframe
                if (app.querySelector('#message')) {
                    const message = app.querySelectorAll('#message')[app.querySelectorAll('#message').length - 1].offsetTop - 68
                    window.scrollTo({ top: message, behavior: 'smooth' })
                }
            }, 2) // <- wait for render (timeout) and then smoothly scroll #app down to the last message
        }
    },
    created() {
        /* If history is enabled, the messages are retrieved from sessionStorage */
        if (this.history() && sessionStorage.getItem('message_history') !== null) {
            this.messages = JSON.parse(sessionStorage.getItem('message_history'))
        }

        /* Session should be persistent (in case of page reload, the context should stay) */
        if (this.history() && sessionStorage.getItem('session') !== null) {
            this.session = sessionStorage.getItem('session')
        }

        else {
            this.session = uuidv1()
            if (this.history()) sessionStorage.setItem('session', this.session)
        }

        /* Cache Agent (this will save bandwith) */
        if (this.history() && sessionStorage.getItem('agent') !== null) {
            this.app = JSON.parse(sessionStorage.getItem('agent'))
        }

        else {
            /* Make GET request to API endpoint */
            let dfDetailsURL = this.config.API_URL + '/get_dialogflow_agent';
            this.$http.get(dfDetailsURL).then(function (dfRes) {
                this.app = dfRes.body;
                if (this.history()) sessionStorage.setItem('agent', JSON.stringify(dfRes.body))
            })
                .catch(error => {
                    this.error = error.message
                })
        }
    },
    methods: {
        send(submission) {
            let request

            /* Text request */
            if (submission.text) {
                request = {
                    session: this.session,
                    queryInput: {
                        text: {
                            text: submission.text,
                            languageCode: this.lang()
                        }
                    }
                }
            }

            /* Audio request */
            else if (submission.audio) {
                request = {
                    session: this.session,
                    queryInput: {
                        audioConfig: {
                            audioEncoding: 'AUDIO_ENCODING_UNSPECIFIED',
                            languageCode: this.lang()
                        }
                    },
                    inputAudio: submission.audio
                }
            }

            this.loading = true
            this.error = null

            /* Make POST request to API endpoint */
            let queryResURL = this.config.API_URL + '/detect_intent';

            this.$http.post(queryResURL, JSON.stringify(request)).then(function (queryResponse) {
                this.messages.push(queryResponse.body)
                this.handle(queryResponse.body) // <- trigger the handle function (explanation below)
            })
                .catch(error => {
                    this.error = error.message
                })
                .then(() => this.loading = false)

        },
        handle(response) {
            /* This function is used for speech output */
            if (response.outputAudio) {
                const mime = this.config.codecs[response.outputAudioConfig.audioEncoding] || this.config.codecs.OUTPUT_AUDIO_ENCODING_UNSPECIFIED

                const base64Audio = (Buffer.from(response.outputAudio.data)).toString('base64');

                const src = `data:${mime};base64,${base64Audio}`
                const output = new Audio(src);

                output.onended = () => this.$refs.input.listen()

                if (!this.muted) output.play()
            }

            else {
                let text // <- init a text variable

                /* Dialogflow Text/SimpleResponses */
                for (const component in response.queryResult.fulfillmentMessages) {
                    if (response.queryResult.fulfillmentMessages[component].text) text = response.queryResult.fulfillmentMessages[component].text.text[0]
                    if (response.queryResult.fulfillmentMessages[component].simpleResponses) text = response.queryResult.fulfillmentMessages[component].simpleResponses.simpleResponses[0].textToSpeech
                    if (response.queryResult.fulfillmentMessages[component].rbmText) text = response.queryResult.fulfillmentMessages[component].rbmText.text
                }

                /* Actions on Google Simple response */
                if (response.queryResult.webhookPayload && response.queryResult.webhookPayload.google) {
                    for (const component in response.queryResult.webhookPayload.google) {
                        if (response.queryResult.webhookPayload.google[component].simpleResponse) text = response.queryResult.webhookPayload.google[component].simpleResponse.textToSpeech
                    }
                }

                const speech = new SpeechSynthesisUtterance(text)
                speech.voiceURI = this.config.voice
                speech.lang = this.lang()
                speech.onend = () => this.$refs.input.listen()

                if (!this.muted) window.speechSynthesis.speak(speech) // <- if app is not muted, speak out the speech
            }

            if (defaultFallbackPhrases.includes(response.queryResult.fulfillmentText)) {
                this.loadNoResponseAnimationModel();
                setTimeout(() => {
                    this.loadBaseModel();
                }, 1500);
            }

            delete response.outputAudio;
        },
        loadBaseModel() {
            const clock = new THREE.Clock();
            const scene = new THREE.Scene();
            const camera = new THREE.PerspectiveCamera(75, 1, 0.1, 1000);

            const renderer = new THREE.WebGLRenderer();
            renderer.setSize(510, 500);
            // renderer.setClearColor(0x000000, 0);
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            // document.getElementById('model-container').removeChild();
            const container = document.getElementById('model-container');
            while (container.firstChild) {
                container.removeChild(container.lastChild);
            }
            container.appendChild(renderer.domElement);

            const material = new THREE.MeshStandardMaterial({
                color: 0x00ff00,
                roughness: 0.5,
                metalness: 0.5,
            });


            const loader = new GLTFLoader();

            loader.load('/idle_animation.glb', function (gltf) {

                const model = gltf.scene;
                model.traverse((child) => {
                    if (child.isMesh) {
                        child.material = material;
                        child.castShadow = true;
                        child.receiveShadow = true;
                    }
                });
                scene.add(model);
                scene.background = new THREE.Color(0x202124);

                const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
                directionalLight.position.set(5, 5, 5);
                directionalLight.castShadow = true; // Enable shadow casting
                scene.add(directionalLight);

                // Set up shadow properties for the light
                directionalLight.shadow.mapSize.width = 1024; // Set these values based on your scene requirements
                directionalLight.shadow.mapSize.height = 1024;
                directionalLight.shadow.camera.near = 0.1;
                directionalLight.shadow.camera.far = 50;

                camera.position.z = 2;
                camera.position.y = 1.2;
                const mixer = new THREE.AnimationMixer(gltf.scene);
                for (let i = 0; i < gltf.animations.length; i++) {
                    const action = mixer.clipAction(gltf.animations[i]);
                    action.play();
                }

                function animate() {

                    requestAnimationFrame(animate);
                    const delta = clock.getDelta();
                    mixer.update(delta);

                    renderer.render(scene, camera);

                }

                animate();

            }, undefined, function (error) {

                console.error(error);

            });
        },
        loadNoResponseAnimationModel() {
            const clock = new THREE.Clock();
            const scene = new THREE.Scene();
            const camera = new THREE.PerspectiveCamera(75, 1, 0.1, 1000);

            const renderer = new THREE.WebGLRenderer();
            renderer.setSize(510, 500);
            // renderer.setClearColor(0x000000, 0);
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            const container = document.getElementById('model-container');
            while (container.firstChild) {
                container.removeChild(container.lastChild);
            }
            container.appendChild(renderer.domElement);

            const material = new THREE.MeshStandardMaterial({
                color: 0x00ff00,
                roughness: 0.5,
                metalness: 0.5,
            });


            const loader = new GLTFLoader();

            loader.load('/no-response-animation.glb', function (gltf) {

                const model = gltf.scene;
                model.traverse((child) => {
                    if (child.isMesh) {
                        child.material = material;
                        child.castShadow = true;
                        child.receiveShadow = true;
                    }
                });
                scene.add(model);
                scene.background = new THREE.Color(0x202124);

                const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
                directionalLight.position.set(5, 5, 5);
                directionalLight.castShadow = true; // Enable shadow casting
                scene.add(directionalLight);

                // Set up shadow properties for the light
                directionalLight.shadow.mapSize.width = 1024; // Set these values based on your scene requirements
                directionalLight.shadow.mapSize.height = 1024;
                directionalLight.shadow.camera.near = 0.1;
                directionalLight.shadow.camera.far = 50;

                camera.position.z = 2;
                camera.position.y = 1.2;
                const mixer = new THREE.AnimationMixer(gltf.scene);
                for (let i = 0; i < gltf.animations.length; i++) {
                    const action = mixer.clipAction(gltf.animations[i]);
                    action.play();
                }

                function animate() {

                    requestAnimationFrame(animate);
                    const delta = clock.getDelta();
                    mixer.update(delta);

                    renderer.render(scene, camera);

                }

                animate();

            }, undefined, function (error) {

                console.error(error);

            });
        }
    }
}
</script>
