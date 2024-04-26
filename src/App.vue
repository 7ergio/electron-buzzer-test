<template>
  <div class="content" :class="{secondState: stateTwo, thirdState: stateThree, fourthState: stateFour, reset: reset, replay: replay}">
    <div class="overlay" :class="{ loaded: imagesLoaded }"></div>

    <div class="location" v-if="chosenLocation">
      <h2 class="location__headline">{{ chosenLocation.heading }}</h2>
      <img :src="chosenLocation.imageSrc" alt="{{ chosenLocation.heading }}">
      <div class="location__description">
        <p>{{ chosenLocation.text }}</p>
        <button class="buzzer__button" @click="playAgain" type="button">Press Buzzer to start</button>
      </div>
    </div>

    <div class="headline" :class="{ loaded: imagesLoaded }">
      <h1><span>Discover<br>All Sides of</span><br>Zürich,<br>Switzerland</h1>
      <button class="buzzer__button" @click="startSecondState" type="button">Press Buzzer to start</button>
    </div>

    <div class="headline-only">
      <h2><span>Discover<br>All Sides of</span><br>Zürich,<br>Switzerland</h2>
    </div>

    <div class="circle-container">
      <div class="circle circle--xl">
        <ul class="circle-wrapper circle-wrapper--xl">
          <li v-for="location in xlCircle" :key="location.id">
            <img :src="location.lqImageSrc" :alt="location.heading">
          </li>
        </ul>
      </div>

      <div class="circle circle--lg">
        <ul class="circle-wrapper circle-wrapper--lg" :class="{spinner: !stateTwo}">
          <li v-for="location in locations" :key="location.id">
            <img :src="location.lqImageSrc" :alt="location.heading">
          </li>
        </ul>
      </div>

      <video class="fire" autoplay loop muted playsinline>
        <source
            src="../public/video/fire-v3.mp4"
            type='video/mp4; codecs="hvc1"'>
        <source
            src="../public/video/fire-v3.webm"
            type="video/webm">
      </video>

      <div class="circle circle--md">
        <ul class="circle-wrapper circle-wrapper--md">
          <li v-for="location in locations" :key="location.id">
            <img :src="location.lqImageSrc" :alt="location.heading">
          </li>
        </ul>
      </div>

      <div class="circle circle--sm">
        <ul class="circle-wrapper circle-wrapper--sm">
          <li v-for="location in smCircle" :key="location.id">
            <img :src="location.lqImageSrc" :alt="location.heading">
          </li>
        </ul>
      </div>

      <div class="circle circle--dot">
        <ul class="circle-wrapper circle-wrapper--dot" :class="{ loaded: imagesLoaded }"></ul>
      </div>
    </div>
  </div>
  <template v-if="stateOne">
    <GlobalEvents @keyup.space="startSecondState"/>
  </template>
  <template v-if="replayButton">
    <GlobalEvents @keyup.space="playAgain"/>
  </template>
</template>

<script>
import { GlobalEvents } from 'vue-global-events'
export default {
  name: 'App',
  components: { GlobalEvents },
  data () {
    return {
      locations: null,
      imagesLoaded: false,
      smCircleLimit: 20,
      xlCircleLimit: 10,
      stateOne: true,
      stateTwo: false,
      stateThree: false,
      stateFour: false,
      reset: false,
      replay: false,
      replayButton: false,
      chosenLocation: null,
      randomIndex: null,
      twoMinTimeoutId: null
    }
  },
  methods: {
    async getLocations() {
      this.locations =  await fetch('data.json')
        .then((res) => {
          if (!res.ok) {
            throw new Error('Network response was not ok')
          }
          return res.json()
        })
        .then((data) => {
          this.imagesLoaded = true
          return data.locations
        });
    },

    spinWheel() {
      const circle = document.querySelector(".circle-wrapper--lg");

      this.randomIndex = Math.floor(Math.random() * this.locations.length);

      const rotationAngle =  (270 - this.randomIndex * (360 / this.locations.length)) + 360 * 7;

      setTimeout(() => (
        circle.style.transform = `rotate(${rotationAngle}deg)`
      ), 100);
      this.chosenLocation = this.locations[this.randomIndex];
    },

    mainResets() {
      this.stateTwo = false
      this.stateThree = false
      this.stateFour = false
      this.chosenLocation = null
      this.randomIndex = null
      document.querySelector(".circle-wrapper--lg").style.transform = null
    },

    startSecondState() {
      this.stateOne = false
      this.stateTwo = true
      this.spinWheel()

      const t = this;
      setTimeout(() => {
        t.stateThree = true
      }, 10000);

      setTimeout(() => {
        t.replayButton = true
      }, 25000);

      this.twoMinTimeoutId = setTimeout(() => {
        if(t.stateThree) {
          t.resetState()
        }
      }, 120000);
    },

    playAgain() {
      this.stateFour = true //animation zoom-out
      this.replay = true
      this.replayButton = false
      clearTimeout(this.twoMinTimeoutId)
      this.twoMinTimeoutId = null

      const t = this;
      setTimeout(() => {
        t.mainResets()
        setTimeout(() => {
          t.startSecondState()
        }, 100);
      }, 6000);
    },

    resetState() {
      this.stateFour = true //animation zoom-out
      this.reset = true
      this.replayButton = false
      const t = this;
      setTimeout(() => {
        t.mainResets()
        t.reset = false
        t.replay = false
        t.stateOne = true
      }, 6000);
    }
  },
  created () {
    this.getLocations()
  },
  computed:{
    smCircle(){
      return this.locations ? this.locations.slice(0, this.smCircleLimit) : this.locations
    },
    xlCircle(){
      return this.locations ? this.locations.slice(0, this.xlCircleLimit) : this.locations
    }
  }
}
</script>

<style lang="scss" scoped>
$circle-size-xl: 145dvw;
$circle-size-lg: 85dvw;
$circle-size-md: 50dvw;
$circle-size-sm: 25dvw;
$circle-size-dot: 2dvw;

$image-size-xl: 18dvw;
$image-size-lg: 7dvw;
$image-size-md: 4dvw;
$image-size-sm: 2dvw;

$smCircleLimit: 20;
$lgCircleLimit: 25;
$xlCircleLimit: 10;

svg {
  width: 0;
  height: 0;
}

@mixin on-circle($item-count, $circle-size, $item-size, $image-width) {
  width:  $circle-size;
  height: $circle-size;
  padding: 0;

  & li {
    display: block;
    position: absolute;
    top:  50%;
    left: 50%;
    margin-top: -(calc($item-size / 4 * 3));
    margin-left: -(calc($item-size / 2));
    z-index: 1;
    width: $image-width;
    overflow: hidden;

  $angle: (calc(360 / $item-count));
  $rot: 0;

    @for $i from 1 through $item-count {
      &:nth-of-type(#{$i}) {
        transform:
            rotate($rot * 1deg)
            translate(calc($circle-size / 2))
            rotate(270deg);
      }

    $rot: $rot + $angle;
    }

    img {
      max-width: $image-width;
    }
  }
}

.content {
  position: relative;
  width:  100dvw;
  height: 100dvh;
  overflow: hidden;
}

.overlay {
  position: absolute;
  width:  100dvw;
  height: 100dvh;
  background-color: #000;
  opacity: 0.3;
  z-index: 1;
}

.headline-only,
.headline {
  position: absolute;
  top:  50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 2;

  &.loaded {
    animation: locationFadeIn 3s ease-in alternate;
  }

  h1,
  h2 {
    font-size: 8dvw;
    line-height: 1.2;

    span {
      background: linear-gradient(to right, #e72f88 20%, #fcc207 40%, #fcc207 60%, #e72f88 80%);
      background-size: 200% auto;
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
      animation: gradient-animation 10s infinite;
    }
  }

  .buzzer__button {
    animation: pulse-animation 4s infinite;
  }
}

.headline-only {
  opacity: 0;
  z-index: 1;
}

.buzzer__button {
  font-size: 2.5dvw;
  font-weight: bold;
  border: none;
  padding: 2.5dvw;
  color: #ffffff;
  background-color: #0278BF;
  width: 100%;
}

.circle-container {
  position: relative;
  width:  100dvw;
  height: 100dvh;
}

.location {
  display: flex;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  opacity: 0;

  &__headline {
    font-size: 4.5dvw;
    margin: 0;
    text-align: center;
    width: 80%;
    position: absolute;
    top: 0;
    left: 50%;
    transform: translateX(-50%);
    opacity: 0;
  }

  &__description {
    position: absolute;
    width: 77%;
    bottom: 0;
    left: 50%;
    transform: translateX(-50%);
    text-align: center;
    opacity: 0;

    p {
      font-size: 2.75dvw;
      font-weight: bold;
      line-height: 1.2;
      margin-bottom: 6dvh;
    }
  }

  .buzzer__button {
    width: 70%;
    opacity: 0;
  }

  img {
    width:  87dvw;
    height: auto;
    margin: 0 auto;
    opacity: 0;
  }
}

.fire {
  position: absolute;
  top:  50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 85dvw;
  height: 85dvw;
  z-index: -1;
  opacity: 0;
}

.circle {
  position: absolute;
  top:  50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;

  &--xl {
    @include on-circle($item-count: $xlCircleLimit, $circle-size: $circle-size-xl, $item-size: $image-size-xl, $image-width: $image-size-xl);
  }

  &--lg {
    @include on-circle($item-count: $lgCircleLimit, $circle-size: $circle-size-lg, $item-size: $image-size-lg, $image-width: $image-size-lg);
  }

  &--md {
    @include on-circle($item-count: $lgCircleLimit, $circle-size: $circle-size-md, $item-size: $image-size-md, $image-width: $image-size-md);
  }

  &--sm {
    @include on-circle($item-count: $smCircleLimit, $circle-size: $circle-size-sm, $item-size: $image-size-sm, $image-width: $image-size-sm);

    img {
      filter: grayscale(40%);
    }
  }

  &--dot {
    @include on-circle($item-count: 0, $circle-size: $circle-size-dot, $item-size: 0, $image-width: 0);
  }

  img {
    display: block;
    transform: rotate(180deg);
    mix-blend-mode: multiply;
  }
}

.circle-wrapper {
  position: relative;
  list-style: none;
  margin: 0;
  padding: 0;

  &--xl {
    width:  $circle-size-xl;
    height: $circle-size-xl;
    animation-name: spin;
    animation-direction: reverse;
    animation-iteration-count: infinite;
    animation-timing-function: linear;
    animation-duration: 30s;
  }

  &--lg {
    width:  $circle-size-lg;
    height: $circle-size-lg;
    transition: 10s;
  }

  &.spinner {
    animation-name: spinWithLightZoom;
    animation-iteration-count: infinite;
    animation-timing-function: linear;
    animation-duration: 10s;
  }

  &--md {
    width:  $circle-size-md;
    height: $circle-size-md;
    animation-name: spin;
    animation-direction: reverse;
    animation-iteration-count: infinite;
    animation-timing-function: linear;
    animation-duration: 15s;
  }

  &--sm {
    width:  $circle-size-sm;
    height: $circle-size-sm;
    animation-name: spinWithZoom;
    animation-iteration-count: infinite;
    animation-timing-function: linear;
    animation-duration: 6s;
  }

  &--dot {
    width:  $circle-size-dot;
    height: $circle-size-dot;
    border-radius: 50%;
    background-color: #000000;

    &.loaded {
      animation-name: impulse-animation;
      animation-timing-function: linear;
      animation-fill-mode: forwards;
      animation-duration: 2s;
    }
  }
}

.secondState {
  .circle-container {
    animation-name: zoomInToTheCenter;
    animation-duration: 10s;
    animation-delay: 2s;
    animation-fill-mode: forwards;
  }

  .circle-wrapper {
    &--xl {
      animation-name: spinMore;
      animation-duration: 5s;

      li {
        animation: blurImageAnimation 10s linear both;
      }
    }

    &--md {
      animation-name: spinMore;
      animation-duration: 5s;

      li {
        animation: blurImageAnimation 10s linear both;
      }
    }

    &--sm {
      animation-name: spinMore;
      animation-duration: 3s;
    }
  }

  .circle-wrapper--dot,
  .headline {
    animation: fadeOutCompletely 1s ease-in alternate;
    animation-fill-mode: forwards;
  }

  &.replay {
    .circle-wrapper--dot,
    .headline {
      animation: none;
      display: none;
    }
  }

  .overlay {
    animation: fadeOutOverlay 10s ease-in alternate;
    animation-fill-mode: forwards;
  }

  .headline__button {
    pointer-events: none;
  }

  .fire {
    animation-name: fadeOutFire;
    animation-duration: 6s;
  }
}

.thirdState {
  .overlay {
    animation: fadeInOverlay 2s ease-in alternate;
    z-index: 3;
  }

  .location {
    opacity: 1;
    z-index: 4;
  }

  .location img {
    animation: imageFadeIn 10s ease-in alternate;
    animation-fill-mode: forwards;
  }

  .location__headline {
    animation: headingFadeIn 0.5s linear alternate;
    animation-delay: 5s;
    animation-fill-mode: forwards;
    z-index: 1;
  }

  .location__description {
    animation: descriptionFadeIn 0.5s linear alternate;
    animation-delay: 10s;
    animation-fill-mode: forwards;
  }

  .location__description .buzzer__button {
    animation: descriptionFadeIn 0.5s linear alternate;
    animation-delay: 15s;
    animation-fill-mode: forwards;
  }
}

.fourthState {

  .location img {
    animation: imageFadeOut 0.5s linear alternate;
    animation-fill-mode: forwards;
  }

  .location__description,
  .location__headline {
    animation: fadeOutCompletely 0.3s linear alternate;
    animation-fill-mode: forwards;
  }

  .circle-container {
    animation-name: zoomOutToTheCenter;
    animation-duration: 6s;
    animation-delay: 0s;
    animation-fill-mode: forwards;
  }

  .circle-wrapper {
    &--xl {
      animation-name: spin;
      li {
        animation: unblurImageAnimation 6s linear both;
      }
    }

    &--lg {
      animation-name: spin;
      animation-iteration-count: infinite;
      animation-timing-function: linear;
      animation-duration: 6s;
      animation-delay: 3s;
    }

    &--md {
      animation-name: spin;
      li {
        animation: unblurImageAnimation 6s linear both;
      }
    }

    &--sm {
      animation-name: spin;
    }
  }

  .headline-only {
    animation: showHeadlineBetweenZooms 6s ease-in alternate;
    animation-fill-mode: forwards;
  }
}

.reset {
  .headline-only {
    animation: none;
  }
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

@keyframes spinWithZoom {
  0% {
    transform: rotate(0deg) scale(1);
  }
  25% {
    transform: rotate(90deg) scale(0.6);
  }
  50% {
    transform: rotate(180deg) scale(1);
  }
  75% {
    transform: rotate(270deg) scale(1.2);
  }
  100% {
    transform: rotate(360deg) scale(1);
  }
}

@keyframes spinWithLightZoom {
  0% {
    transform: rotate(0deg) scale(1);
  }
  25% {
    transform: rotate(90deg) scale(1.05);
  }
  50% {
    transform: rotate(180deg) scale(1);
  }
  75% {
    transform: rotate(270deg) scale(0.9);
  }
  100% {
    transform: rotate(360deg) scale(1);
  }
}

@keyframes blurImageAnimation {
  0% {
    filter: blur(0);
  }
  100% {
    filter: blur(1px);
    box-shadow: inset 24px 0 30px -4px rgba(0, 0, 0, 0.8),
    inset -24px 0 30px -4px rgba(0, 0, 0, 0.8);
  }
}

@keyframes unblurImageAnimation {
  0% {
    filter: blur(1px);
    box-shadow: inset 24px 0 30px -4px rgba(0, 0, 0, 0.8),
    inset -24px 0 30px -4px rgba(0, 0, 0, 0.8);
  }
  100% {
    filter: blur(0);
    box-shadow: none;
  }
}

@keyframes fadeOutCompletely {
  from {
    opacity: 1;
  }
  to {
    opacity: 0;
  }
}

@keyframes fadeOut {
  from {
    opacity: 1;
  }
  to {
    opacity: 0.5;
  }
}

@keyframes fadeIn {
  from {
    opacity: 0.5;
  }
  to {
    opacity: 1;
  }
}

@keyframes fadeOutOverlay {
  from {
    opacity: 0.3;
  }
  to {
    opacity: 0;
  }
}

@keyframes fadeInOverlay {
  from {
    opacity: 0;
  }
  to {
    opacity: 0.3;
  }
}

@keyframes locationFadeIn {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

@keyframes showHeadlineBetweenZooms {
  0% {
    opacity: 0;
  }
  25% {
    opacity: 0;
  }
  50% {
    opacity: 0;
  }
  75% {
    opacity: 1;
  }
  90% {
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}

@keyframes fadeOutFire {
  0% {
    opacity: 1;
  }
  90% {
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}

@keyframes imageFadeIn {
  0% {
    filter: brightness(1);
    width: 60dvw;
    opacity: 0;
  }
  1% {
    filter: brightness(1);
    width: 60dvw;
    opacity: 1;
  }
  10% {
    filter: brightness(1);
    width:  87dvw;
    opacity: 1;
  }
  80% {
    filter: brightness(1);
    width:  87dvw;
    opacity: 1;
  }
  90% {
    filter: brightness(0.8);
    width:  87dvw;
    opacity: 1;
  }
  100% {
    filter: brightness(0.6);
    width:  87dvw;
    opacity: 1;
  }
}

@keyframes imageFadeOut {
  0% {
    width: 87dvw;
    opacity: 1;
    filter: brightness(0.6);
  }
  10% {
    width: 95dvw;
    opacity: 1;
    filter: brightness(0.7);
  }
  90% {
    width: 60dvw;
    opacity: 1;
    filter: brightness(0.7);
  }
  100% {
    width: 60dvw;
    opacity: 0;
    filter: brightness(0.7);
  }
}

@keyframes headingFadeIn {
  0% {
    top: 0;
    opacity: 0;
  }
  100% {
    top: 5%;
    opacity: 1;
  }
}

@keyframes descriptionFadeIn {
  0% {
    bottom: 0;
    opacity: 0;
  }
  100% {
    bottom: 5%;
    opacity: 1;
  }
}

@keyframes spinMore {
  0% {
    transform: rotate(360deg);
  }
  100% {
    transform: rotate(1000deg);
  }
}

@keyframes zoomInToTheCenter {
  0% {
    transform: scale(1);
    transform-origin: 50% var(--zoom-scale);
  }
  100% {
    transform: scale(8);
    transform-origin: 50% var(--zoom-scale);
  }
}

@keyframes zoomOutToTheCenter {
  0% {
    transform: scale(8);
    transform-origin: 50% var(--zoom-scale);
  }
  10% {
    transform: scale(8);
    transform-origin: 50% var(--zoom-scale);
  }
  100% {
    transform: scale(1);
    transform-origin: 50% 50%;
  }
}

@keyframes impulse-animation {
  0% {
    transform: scale(1);
    opacity: 0.5;
  }
  100% {
    transform: scale(100);
    opacity: 0;
  }
}

@keyframes pulse-animation {
  0% {
    box-shadow: 0 0 0 0 rgba(231, 47, 136, 0.5);
  }
  30% {
    box-shadow: 0 0 0 5dvw rgba(231, 41, 146, 0);
  }
  100% {
    box-shadow: 0 0 0 5dvw rgba(231, 41, 146, 0);
  }
}

@keyframes gradient-animation {
  to {
    background-position: 200% center;
  }
}

</style>