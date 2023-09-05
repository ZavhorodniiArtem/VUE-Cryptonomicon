<template>
  <button
    type="button"
    :disabled="disabled"
    class="btn flex"
    :class="{ 'opacity-30': disabled }"
  >
    <p class="mr-2">Add</p>
    <div>
      <svg
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke-width="1.5"
        stroke="currentColor"
        class="w-6 h-6"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          d="M12 9v6m3-3H9m12 0a9 9 0 11-18 0 9 9 0 0118 0z"
        />
      </svg>
    </div>
  </button>
</template>

<script>
export default {
  props: {
    disabled: {
      type: Boolean,
      required: false,
      default: false,
    },
  },
};
</script>

<style scoped>
.btn {
  border-radius: 16px;
  padding: 4px 16px;
  background: transparent;
  border: 2px solid lightseagreen;
  cursor: pointer;
  margin-left: 16px;
  height: fit-content;
}

.btn:hover {
  background: lightseagreen;
  color: white;
}
</style>
