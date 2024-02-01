<template>
  <h1 class="relate">{{ msg }}
    <div class="author">by Orlando Sarceda Jr.</div>
  </h1>
</template>

<script>
export default {
  name: 'Title',
  props: {
    msg: String
  },
}
</script>
