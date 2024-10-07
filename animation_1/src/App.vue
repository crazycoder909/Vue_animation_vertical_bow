<template>
  <div class="draggable"  ref="draggableDiv">
    <svg class="bg">
      <defs>
        <filter id="shadow1" x="-20%" y="-20%" width="140%" height="140%">
          <feDropShadow dx="1" dy="1" stdDeviation="2" :flood-color="shadowColor1" />
        </filter>
        <filter id="shadow2" x="-20%" y="-20%" width="140%" height="140%">
          <feDropShadow dx="1" dy="1" stdDeviation="2" :flood-color="shadowColor2" />
        </filter>
      </defs>
      <path :d="headerPath1" fill="#37C7DA" filter="url(#shadow1)"></path>
      <path :d="headerPath1_1" fill="#846DFD" ></path>
      <path :d="headerPath2" fill="#F7EE52" filter="url(#shadow2)"></path>
      <path :d="headerPath2_1" fill="#2BCFD6" ></path>
    </svg>
  </div>
</template>

<script setup>
import { reactive, computed, onMounted, onUnmounted, ref } from 'vue';
import dynamics from 'dynamics.js';

const headerHeight = 40; // Original height of the header
const maxMovement = 15; // Maximum vertical movement
const animationSpeed = 700; // Speed of the animation

const draggableDiv = ref(null);
const divWidth = ref(0);
const divHeight = ref(0);

onMounted(() => {
  // Check if draggableDiv is not null before accessing properties
  if (draggableDiv.value) {
    divWidth.value = draggableDiv.value.clientWidth;  // Width in pixels
    divHeight.value = draggableDiv.value.clientHeight; // Height in pixels

    console.log('Width of the div:', divWidth);  // Log the width
    console.log('Height of the div:', divHeight); // Log the height
  } else {
    console.error('draggableDiv is null');
  }
  // startAnimation(); // Start the animation loop
  animateHeight(); // Start the height animation

});

const c = reactive({ y: headerHeight }); // Reactive y position
const currentHeight = reactive({ value: 60 }); // Reactive height variable

const shadowColor1 = computed(() => '#37C7DA'); // Same color as path 1
const shadowColor2 = computed(() => '#F7EE52'); // Same color as path 2

const headerPath1 = computed(() => { 
  const thinHeaderHeight = 0; // New thinner height for the blue part
  return `M0,0 L0,${(divHeight.value)/2} ${thinHeaderHeight},${(divHeight.value/2)} Q${c.y},${currentHeight.value} ${thinHeaderHeight},0`; 
});

const headerPath1_1 = computed(() => {
  const thinHeaderHeight = 0; // New thinner height for the blue part
  return `M0,0 L0,${divHeight.value / 2} ${thinHeaderHeight},${divHeight.value / 2} Q${c.y - 20},${currentHeight.value} ${thinHeaderHeight},0`;
});

const headerPath2 = computed(() => {
  const thinHeaderHeight = 0; // New thinner height for the blue part
  return `M${divWidth.value},0 L${divWidth.value},${divHeight.value / 2} ${divWidth.value},${divHeight.value / 2} Q${divWidth.value - c.y},${currentHeight.value} ${divWidth.value},0`;
});

const headerPath2_1 = computed(() => {
  const thinHeaderHeight = 0; // New thinner height for the blue part
  return `M${divWidth.value},0 L${divWidth.value},${divHeight.value / 2} ${divWidth.value},${divHeight.value / 2} Q${divWidth.value - c.y + 20},${currentHeight.value} ${divWidth.value},0`;
});


// Calculate the highest position of the animation
const highestPosition = computed(() => {
  return {
    x: 0, // Since there's no horizontal movement
    y: headerHeight - maxMovement // Calculate the highest y position
  };
});

let direction = 1; // 1 for down, -1 for up

// Function to animate height between 60 and 200 and back to 60
function animateHeight() {
  const startHeight = 60;
  const endHeight = 200;
  const frameRate = 60; // 60 frames per second
  const animationDuration = 2500;
  const totalFrames = (animationDuration / 1000) * frameRate;
  const heightIncrement = (endHeight - startHeight) / (totalFrames / 2); // Increment for upward movement
  let frameCount = 0;

  function updateHeight() {
    if (frameCount < totalFrames / 2) { // First half (increasing)
      currentHeight.value += heightIncrement;
    } else if (frameCount < totalFrames) { // Second half (decreasing)
      currentHeight.value -= heightIncrement;
    } else {
      frameCount = -1; // Reset frame count for next cycle
    }
    frameCount++;
    
    requestAnimationFrame(updateHeight); // Use requestAnimationFrame for smoother animation
  }

  updateHeight(); // Start the animation
}


function startAnimation() {
  const targetY = c.y + direction * maxMovement;

  console.log(c.x)
  dynamics.animate(
    c,
    { y: targetY },
    {
      type: dynamics.spring,
      duration: animationSpeed,
      friction: 100,
      tension: 100,
      complete: () => {
        direction *= -1; // Reverse direction after reaching target
        startAnimation(); // Continue the animation
      }
    }
  );
}

onUnmounted(() => {
  dynamics.stop(c); // Stop any ongoing dynamics animation on unmount
});
</script>

<style scoped>
.draggable {
  background-color: #012438;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
  width: 240px; /* Adjusted width */
  height: 420px; /* Adjusted height */
  overflow: hidden;
  margin: 10px auto;
  position: relative;
  text-align: center;
  font-size: 14px;
  font-weight: 300;
  user-select: none;
  border-radius: 8px;
}

.bg {
  margin-top: 20px;
  position: absolute;
  height: 220px;
  top: 0;
  left: 0;
  z-index: 0;
}

.position-info {
  margin-top: 20px; /* Space for the position info */
  font-size: 12px;
  color: #555;
}
</style>
