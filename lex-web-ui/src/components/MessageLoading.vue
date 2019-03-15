<template>
  <v-flex d-flex class="message message-bot">
    <!-- contains message and response card -->
    <v-layout column ma-2 class="message-layout">

      <!-- contains message bubble and date -->
      <v-flex d-flex class="message-bubble-date-container">
        <v-layout column class="message-bubble-column">

          <!-- contains message bubble and avatar -->
          <v-flex d-flex class="message-bubble-avatar-container">
            <v-layout row class="message-bubble-row">
              <div
                class="message-bubble focusable"
              >
                <div class="progress">
                  <div class="bounce1"></div>
                  <div class="bounce2"></div>
                  <div class="bounce3"></div>

                </div>
              </div>
            </v-layout>
          </v-flex>
        </v-layout>
      </v-flex>
    </v-layout>
  </v-flex>
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

export default {
  name: 'messageLoading',
  data() {
    return {
      progress: '.',
    };
  },
  computed: {
  },
  methods: {
  },
  created() {
    this.interval = setInterval(() => {
      if (this.progress.length > 2) {
        this.progress = '.';
      } else {
        this.progress += '.';
      }
    }, 500);
  },
  destroyed() {
    clearInterval(this.interval);
  },
};
</script>

<style scoped>
.message, .message-bubble-column {
  flex: 0 0 auto;
}

.message, .message-bubble-row {
  max-width: 80vw;
}

.message-bubble {
  border-radius: 24px;
  display: inline-flex;
  font-size: calc(1em + 0.25vmin);
  padding: 0 12px;
  width: fit-content;
  align-self: center;
}


.message-bot .message-bubble {
  /*background-color: #fff; !* red-50 from material palette *!*/
}
.progress {
  margin: 30px auto 0;
  width: 70px;
  text-align: center;
}

.progress > div {
  width: 18px;
  height: 18px;
  background-color: #333;

  border-radius: 100%;
  display: inline-block;
  -webkit-animation: sk-bouncedelay 1.4s infinite ease-in-out both;
  animation: sk-bouncedelay 1.4s infinite ease-in-out both;
}

.progress .bounce1 {
  -webkit-animation-delay: -0.32s;
  animation-delay: -0.32s;
}

.progress .bounce2 {
  -webkit-animation-delay: -0.16s;
  animation-delay: -0.16s;
}
@-webkit-keyframes sk-bouncedelay {
  0%, 80%, 100% { -webkit-transform: scale(0) }
  40% { -webkit-transform: scale(1.0) }
}

@keyframes sk-bouncedelay {
  0%, 80%, 100% {
    -webkit-transform: scale(0);
    transform: scale(0);
  } 40% {
      -webkit-transform: scale(1.0);
      transform: scale(1.0);
    }
}

</style>
