<template>
    <transition
      name="fade"
      @before-enter="beforeEnter"
      @enter="enter"
      @leave="leave"
    >
      <nuxt />
    </transition>
  </template>
  
  <script>
  export default {
    methods: {
      beforeEnter(el) {
        el.style.opacity = 0;
      },
      enter(el, done) {
        el.offsetHeight; // trigger reflow
        el.style.transition = 'opacity 0.5s';
        el.style.opacity = 1;
        done();
      },
      leave(el, done) {
        el.style.transition = 'opacity 0.5s';
        el.style.opacity = 0;
        done();
      },
    },
  };
  </script>
  