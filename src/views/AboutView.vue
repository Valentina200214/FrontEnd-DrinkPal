<template>
  <div class="video-container">
    <!-- Usando require para que Webpack procese el video -->
    <video controls width="40%" height="auto">
      <source :src="require('@/assets/tf.mp4')" type="video/mp4">
      Tu navegador no soporta videos HTML5.
    </video>
  </div>
</template>

<script>
export default {
  name: 'VideoLocal'
}
</script>
