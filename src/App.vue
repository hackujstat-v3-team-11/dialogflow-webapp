<template>
<div id="app">
<section class="hero is-fullheight">

        <div class="hero-head">
          <header class="hero is-link is-bold">
            <div class="hero-body">
              <div class="container">
                <p class="title">
                  Statistika hrou
                </p>
                <p class="subtitle">
                  Dialogflow WebApp
                </p>
              </div>
            </div>
          </header>
        </div>

        <div class="hero-body">
            <div  class="chat-window">
                <div v-for="(a, aIndex) in answers" v-bind:key="aIndex" class="p">
                    <span class='tag is-medium'>{{a.result.resolvedQuery}}</span>

                    <span v-if="a.result.fulfillment.spemessagesech" class='tag is-medium'> {{a.result.fulfillment.speech}}</span>

                    <div v-for="(m, mIndex) in a.result.fulfillment.messages" v-bind:key="mIndex" class="p">
                        <div v-if="m.type === 2">
                            <div style="white-space: pre;">{{m.title}}</div>
                            <div v-if="m.replies" class="field is-grouped replies">
                            <b-button v-for="(r, rIndex) in m.replies" v-bind:key="rIndex" type="is-link" v-on:click="autosubmit(r)">{{r}}</b-button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="hero-foot">
          <footer class="section is-small">
            <form v-on:submit.prevent="submit">
                <div class="field has-addons">
                <div class="control">
                                    <a v-on:click="microphone(true)"><b-icon icon="microphone"></b-icon></a>
                    <a v-on:click="mute(true)" v-if="muted == false"><b-icon icon="volume-source"></b-icon></a>
                    <a v-on:click="mute(false)" v-else><b-icon icon="volume-off"></b-icon></a>
                </div>
                <div class="control is-expanded">

                    <b-input
                            v-model="query"
                           :placeholder="queryTitle" autofocus
                        >
                        </b-input>
                </div>
                <div class="control">
                    <button class="button is-info">
                    Send
                    </button>
                </div>
                </div>
            </form>
          </footer>
        </div>
      </section>


</div>
</template>

<script>
import { ApiAiClient } from 'api-ai-javascript'

const client = new ApiAiClient({accessToken: '166b02e660ca4415a425afa5b1ade086'})
const speechLang = "en-GB"; // output language
const recognitionLang = "en-US"; // input(recognition) language
const googleIt = true; // ask users to google their request, in case of input.unknown action

export default {
    name: 'app',
    data: () => {
        return {
            answers: [],
            query: '',
            speech: '',
            micro: false,
            muted: false,
            online: navigator.onLine,
            queryTitle: "Ask me something...",
        }
    },
    watch: {
        answers() {
            // setTimeout(() => {
            //     document.querySelector('#bottom').scrollIntoView({
            //         behavior: 'smooth'
            //     })
            // }, 2) // if new answers arrive, wait for render and then smoothly scroll down to #bottom selector, used as anchor
        }
    },
    methods: {
        submit(){
            if (this.query === "Go ahead, im listening..."){
                this.query = '';
            }
            if (!this.query || this.query.length < 1) return;

            client.textRequest(this.query).then((response) => {
                if(response.result.action === "input.unknown" && googleIt === true){
                    response.result.fulfillment.messages[0].unknown = true
                    response.result.fulfillment.messages[0].text = response.result.resolvedQuery
                } // if the googleIt is enabled, show the button

                this.answers.push(response)
                this.handle(response) // handle the response in handle() method

                this.query = ''
                this.speech = "Go ahead, im listening..." // reset query and speech
            })
        },
        handle(response){
            if(response.result.fulfillment.speech || response.result.fulfillment.messages[0].type == 'simple_response'){
                let speech = new SpeechSynthesisUtterance(response.result.fulfillment.speech || response.result.fulfillment.messages[0].textToSpeech)
                speech.voiceURI = 'native'
                speech.lang = speechLang;

                if(this.muted == false) window.speechSynthesis.speak(speech) // Speech output if microphone is allowed
            }
        },
        autosubmit(suggestion){
            this.query = suggestion
            this.submit()
        },
        mute(mode){
            this.muted = mode
        },
        microphone(mode){
            this.micro = mode
            let self = this // correct scope

            if(mode == true){
                this.queryTitle = "Go ahead, im listening...";
                let recognition = new window.webkitSpeechRecognition // chrome speech recognition

                recognition.interimResults = true;
                recognition.lang = recognitionLang;
                recognition.start()

                recognition.onresult = function(event){
                    for (var i = event.resultIndex; i < event.results.length; ++i){
                        self.speech = event.results[i][0].transcript
                    }
                }

                recognition.onend = function(){
                    recognition.stop()
                    self.queryTitle = "Go ahead, im listening..."
                    self.micro = false
                    self.autosubmit(self.speech)
                }
            }
        }
    }
}
</script>

<style>
#app .chat-window {
    height: 100%;
    width: 100%;
    position: relative;
}
#app .chat-window p {
    padding: 0.25em;
    text-align: left;
    overflow-wrap: normal;
}
#app .chat-window .p {
    width: 100%;
    padding: 0.25em;
    text-align: left;
    overflow-wrap: normal;
}
.replies button {
    margin-right: 0.2em;
}
</style>
