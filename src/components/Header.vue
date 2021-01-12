<template>
  <header class="header">
    <div class="header__container">
      <div class="header__logo">
        PageSpeed Dashboard
      </div>
    </div>
  </header>
</template>

<style lang="scss">
  .header {
    padding: $px16 0;

    &__logo {
      font-weight: 900;
    }

    &__container {
      width: $px1024;
      margin: 0 auto;
      display: flex;
      align-items: center;
      justify-content: space-between;
    }

    &__menu {
      display: block;

      list-style: none;
    }
  }
</style>
