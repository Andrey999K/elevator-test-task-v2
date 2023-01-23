<template>
    <div class="mine">
          <div v-for="floor in floors" :key="floor.numberFloor" class="floor"></div>
          <div class="elevator" 
            v-bind:class="{'elevator-pause' : mine.stoped}" 
            v-bind:style="{
              bottom: `${mine.currentHeight}%`, 
              height: `calc(100% / ${this.floors.length})`
            }">
            <div class="elevator__indication" v-bind:class=mine.direction>
              <div class="elevator__indication-direction">
                <svg class="arrow-up" width="12" height="11" viewBox="0 0 12 11" fill="none" xmlns="http://www.w3.org/2000/svg">
                  <path d="M6.56569 0.434315C6.25327 0.121895 5.74673 0.121895 5.43431 0.434315L0.343146 5.52548C0.0307262 5.8379 0.0307262 6.34443 0.343146 6.65685C0.655565 6.96927 1.1621 6.96927 1.47452 6.65685L6 2.13137L10.5255 6.65685C10.8379 6.96927 11.3444 6.96927 11.6569 6.65685C11.9693 6.34443 11.9693 5.8379 11.6569 5.52548L6.56569 0.434315ZM6.8 11L6.8 1H5.2L5.2 11H6.8Z" fill="white"/>
                </svg>
                <svg  class="arrow-down" width="12" height="11" viewBox="0 0 12 11" fill="none" xmlns="http://www.w3.org/2000/svg">
                  <path d="M5.43431 10.5657C5.74673 10.8781 6.25327 10.8781 6.56569 10.5657L11.6569 5.47452C11.9693 5.1621 11.9693 4.65557 11.6569 4.34315C11.3444 4.03073 10.8379 4.03073 10.5255 4.34315L6 8.86863L1.47452 4.34315C1.1621 4.03073 0.655565 4.03073 0.343146 4.34315C0.0307262 4.65557 0.0307262 5.1621 0.343146 5.47452L5.43431 10.5657ZM5.2 0L5.2 10H6.8L6.8 0L5.2 0Z" fill="white"/>
                </svg>
              </div>
              <span class="elevator__indication-floor">{{ mine.onFloor }}</span>
            </div>
          </div>
        </div>
</template>

<script>
export default {
    props: ['floors', 'mine']
}
</script>