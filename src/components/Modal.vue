<template>  
  <div class="modal is-active">
	  <div class="modal-background"></div>
	  <div class="modal-card">
	    <header class="modal-card-head">
        <p class="is-size-4-desktop is-size-5-touch modal-card-title"><slot name="title"></slot></p>
	      <button class="delete" @click="$emit('close')" aria-label="close"></button>
	    </header>
      <section class="modal-card-body">
        <slot></slot>
      </section>
    </div>
	</div>
</template>

<script>

export default {
  name: 'Modal',

}

</script>

<style scoped>
.seperator {
  margin: 1em 0;
  height: 2px;
  background-color: #dbdbdb;
  
}

.modal-card-title {
  max-width: 80%;
  margin-right: 17%;
}
</style>
