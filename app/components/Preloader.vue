<template>
  <div class="preloader" :data-phase="phase">
    <ClientOnly>
      <div class="preloader-counter">
        <span :data-value="Math.floor(Math.abs(count) / 100) % 10">
          {{ Math.floor(Math.abs(count) / 100) % 10 }}
        </span>
        <span :data-value="Math.floor(Math.abs(count) / 10) % 10" v-if="count > 9">
          {{ Math.floor(Math.abs(count) / 10) % 10 }}
        </span>
        <span :data-value="Math.floor(Math.abs(count) / 1) % 10">
          {{ Math.floor(Math.abs(count) / 1) % 10 }}
        </span>
      </div>
    </ClientOnly>
    <div class="preloader-info">2022 Creator Wrapped is complete and this is your data</div>
    <div class="preloader-year">
      <span v-for="_ in 3">2022</span>
    </div>
    <div class="preloader-tag">
      <span v-for="_ in 11">#SPOTIFYWRAPPED</span>
    </div>
  </div>
</template>

<script setup lang="ts">
import gsap from "gsap";
import * as THREE from "three";
import { getProject, types } from "@theatre/core";
import studio from "@theatre/studio";
studio.initialize();

const count = ref(0);
const target = ref(0);
const phase = ref(0);

onMounted(async () => {
  const assets: Array<String> = [];
  const images = document.querySelectorAll("img");

  const project = getProject("Spotify Wrapped");
  const sheet = project.sheet("Preloader Curtain");
  const years = gsap.utils.toArray<HTMLElement>(".preloader-year span")!;
  const tags = gsap.utils.toArray<HTMLElement>(".preloader-tag span")!;
  const $years = years.map((_, i) => {
    return sheet.object(`Year${i + 1}`, {
      x: 1512,
      y: 100,
      opacity: types.number(1, { range: [0, 1] }),
    });
  });
  const $tags = tags.map((_, i) => {
    return sheet.object(`Tag${i + 1}`, {
      x: 1512,
      y: 100,
      opacity: types.number(1, { range: [0, 1] }),
    });
  });

  years.forEach((year, i) => {
    $years[i]?.onValuesChange(($year) => {
      year.style.transform = `translateX(${$year.x}px) translateY(${$year.y}px)`;
      year.style.opacity = `${$year.opacity}`;
    });
  });

  tags.forEach((tag, i) => {
    $tags[i]?.onValuesChange(($tag) => {
      tag.style.transform = `translateX(${$tag.x}px) translateY(${$tag.y}px)`;
      tag.style.opacity = `${$tag.opacity}`;
    });
  });

  images.forEach((img) => {
    assets.push(img.src);
  });

  for (let i = 0; i < assets.length; i++) {
    await fetch(assets[i] as RequestInfo);
    // await gsap.delayedCall((Math.random() * i) / 4, update);
    update();
    target.value += 100 / images.length;
  }

  function onPreloaded() {
    gsap
      .timeline({ delay: 0.5 })
      .fromTo(
        ".preloader-counter span",
        { yPercent: 0, clipPath: "inset(0% 0% 0% 0%)" },
        { yPercent: -85, clipPath: "inset(85% 0% 0% 0%)", stagger: 0.0875, duration: 0.675 },
      )
      .to(".preloader-info", { opacity: 0, duration: 0.2, delay: 0.2 });
  }

  function update() {
    if (count.value < target.value) {
      // count.value += 2.5 / 60;
      count.value += 25 / 60;
    }
    if (phase.value === 0 && count.value > 25) {
      phase.value = 1;
    }
    if (phase.value === 1 && count.value > 50) {
      phase.value = 2;
    }
    if (phase.value === 2 && count.value > 75) {
      phase.value = 3;
    }

    if (count.value >= 100) {
      onPreloaded();
      return;
    }
    requestAnimationFrame(update);
  }
});
</script>
