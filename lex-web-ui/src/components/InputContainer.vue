<template>
  <v-footer app fixed>
    <v-layout
      row
      justify-center
      class="input-container"
    >
      <v-toolbar id="input-toolbar" flat dense>
        <!--
          using v-show instead of v-if to make recorder-status transition work
        -->
        <v-text-field
          v-bind:label="textInputPlaceholder"
          v-show="shouldShowTextInput"
          v-model="textInput"
          v-on:keyup.enter.stop="postTextMessage"
          v-on:focus="onTextFieldFocus"
          v-on:blur="onTextFieldBlur"
          id="text-input"
          name="text-input"
          single-line
          class="osu-solo"
          hide-details
        ></v-text-field>

        <recorder-status
          v-show="!shouldShowTextInput"
        ></recorder-status>

        <!-- separate tooltip as a workaround to support mobile touch events -->
        <!-- tooltip should be before btn to avoid right margin issue in mobile -->
        <v-tooltip
          activator=".input-button"
          v-model="shouldShowTooltip"
          ref="tooltip"
          left
        >
          <span id="input-button-tooltip">{{inputButtonTooltip}}</span>
        </v-tooltip>
        <v-btn
          v-if="shouldShowSendButton"
          v-on:click="postTextMessage"
          v-on="tooltipEventHandlers"
          v-bind:disabled="isSendButtonDisabled"
          ref="send"
          class="beaver--text input-button"
          icon
        >
          <v-icon medium>arrow_forward</v-icon>
        </v-btn>
        <v-btn
          v-else
          v-on:click="onMicClick"
          v-on="tooltipEventHandlers"
          v-bind:disabled="isMicButtonDisabled"
          ref="mic"
          class="black--text input-button"
          icon
        >
          <v-icon medium>{{micButtonIcon}}</v-icon>
        </v-btn>
      </v-toolbar>
    </v-layout>
  </v-footer>
</template>

<script>
/*
Copyright 2017-2017 Amazon.com, Inc. or its affiliates. All Rights Reserved.

Licensed under the Amazon Software License (the "License"). You may not use this file
except in compliance with the License. A copy of the License is located at

http://aws.amazon.com/asl/

or in the "license" file accompanying this file. This file is distributed on an "AS IS"
BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, express or implied. See the
License for the specific language governing permissions and limitations under the License.
*/
/* eslint no-console: ["error", { allow: ["warn", "error"] }] */

import RecorderStatus from '@/components/RecorderStatus';

export default {
  name: 'input-container',
  data() {
    return {
      textInput: '',
      isTextFieldFocused: false,
      shouldShowTooltip: false,
      // workaround: vuetify tooltips doesn't seem to support touch events
      tooltipEventHandlers: {
        mouseenter: this.onInputButtonHoverEnter,
        mouseleave: this.onInputButtonHoverLeave,
        touchstart: this.onInputButtonHoverEnter,
        touchend: this.onInputButtonHoverLeave,
        touchcancel: this.onInputButtonHoverLeave,
      },
    };
  },
  props: ['textInputPlaceholder', 'initialSpeechInstruction'],
  components: {
    RecorderStatus,
  },
  computed: {
    isBotSpeaking() {
      return this.$store.state.botAudio.isSpeaking;
    },
    isSpeechConversationGoing() {
      return this.$store.state.recState.isConversationGoing;
    },
    isMicButtonDisabled() {
      return this.isMicMuted;
    },
    isMicMuted() {
      return this.$store.state.recState.isMicMuted;
    },
    isRecorderSupported() {
      return this.$store.state.recState.isRecorderSupported;
    },
    isRecorderEnabled() {
      return this.$store.state.recState.isRecorderEnabled;
    },
    isSendButtonDisabled() {
      return this.textInput.length < 1;
    },
    micButtonIcon() {
      if (this.isMicMuted) {
        return 'mic_off';
      }
      if (this.isBotSpeaking || this.isSpeechConversationGoing) {
        return 'stop';
      }
      return 'mic';
    },
    inputButtonTooltip() {
      if (this.shouldShowSendButton) {
        return 'send';
      }
      if (this.isMicMuted) {
        return 'mic seems to be muted';
      }
      if (this.isBotSpeaking || this.isSpeechConversationGoing) {
        return 'interrupt';
      }
      return 'click to use voice';
    },
    shouldShowSendButton() {
      return (
        (this.textInput.length && this.isTextFieldFocused) ||
        (!this.isRecorderSupported || !this.isRecorderEnabled)
      );
    },
    shouldShowTextInput() {
      return !(this.isBotSpeaking || this.isSpeechConversationGoing);
    },
  },
  methods: {
    onInputButtonHoverEnter() {
      this.shouldShowTooltip = true;
    },
    onInputButtonHoverLeave() {
      this.shouldShowTooltip = false;
    },
    onMicClick() {
      this.onInputButtonHoverLeave();
      if (this.isBotSpeaking || this.isSpeechConversationGoing) {
        return this.$store.dispatch('interruptSpeechConversation');
      }
      if (!this.isSpeechConversationGoing) {
        return this.startSpeechConversation();
      }

      return Promise.resolve();
    },
    onTextFieldFocus() {
      this.isTextFieldFocused = true;
    },
    onTextFieldBlur() {
      if (!this.textInput.length && this.isTextFieldFocused) {
        this.isTextFieldFocused = false;
      }
    },
    playInitialInstruction() {
      const isInitialState = ['', 'Fulfilled', 'Failed']
        .some(initialState => (
          this.$store.state.lex.dialogState === initialState
        ));

      return (isInitialState) ?
        this.$store.dispatch(
          'pollySynthesizeSpeech',
          this.initialSpeechInstruction,
        ) :
        Promise.resolve();
    },
    postTextMessage() {
      this.onInputButtonHoverLeave();
      this.textInput = this.textInput.trim();
      // empty string
      if (!this.textInput.length) {
        return Promise.resolve();
      }

      const message = {
        type: 'human',
        text: this.textInput,
      };

      return this.$store.dispatch('postTextMessage', message)
        .then(() => {
          this.textInput = '';
        });
    },
    startSpeechConversation() {
      if (this.isMicMuted) {
        return Promise.resolve();
      }
      return this.setAutoPlay()
        .then(() => this.playInitialInstruction())
        .then(() => this.$store.dispatch('startConversation'))
        .catch((error) => {
          console.error('error in startSpeechConversation', error);
          const errorMessage = (this.$store.state.config.ui.showErrorDetails) ?
            ` ${error}` : '';

          this.$store.dispatch(
            'pushErrorMessage',
            "Sorry, I couldn't start the conversation. Please try again." +
            `${errorMessage}`,
          );
        });
    },
    /**
     * Set auto-play attribute on audio element
     * On mobile, Audio nodes do not autoplay without user interaction.
     * To workaround that requirement, this plays a short silent audio mp3/ogg
     * as a reponse to a click. This silent audio is initialized as the src
     * of the audio node. Subsequent play on the same audio now
     * don't require interaction so this is only done once.
     */
    setAutoPlay() {
      if (this.$store.state.botAudio.autoPlay) {
        return Promise.resolve();
      }
      return this.$store.dispatch('setAudioAutoPlay');
    },
  },
};
</script>
<style>
  .footer {
    /* make footer same height as dense toolbar */
    min-height: 48px;
    margin-bottom: 1em;
    background-color: transparent !important;
  }
  #input-toolbar {
    max-width: 768px;
    background-color: transparent !important;
  }
  .beaver--text {
    color: #D73F09 !important;
    margin-right: 8px !important;
  }
  .osu-solo {
    border: 1px solid #E9E5E4;
    box-sizing: border-box;
    border-radius: 8px;
    background-color: #fff;
    min-height: 48px;
    margin-left: 16px !important;
  }
  .osu-solo label {
    top: 10px !important;
    padding: 0 20px;
    color: #8f8582 !important;
  }
  .toolbar .input-group__details {
    display: none;
  }
  .osu-solo #text-input {
    height: auto;
    padding: 0 20px;
  }
  @media only screen and (max-width: 350px) {
    .osu-solo {
      margin-left: 0 !important;
    }
  }
</style>
