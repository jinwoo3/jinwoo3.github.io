<template>
  <div class="nav">
    <b-navbar class="container nav">
      <template slot="brand">
        <b-navbar-item tag="router-link" :to="{ path: '/' }">
          <img src="@/assets/images/soomgo.png" alt="Soomgo" />
        </b-navbar-item>
      </template>
      <template slot="start">
        <b-navbar-item to="/about" tag="router-link">
          About
        </b-navbar-item>
        <b-navbar-item href="https://soomgo.recruitee.com/">
          Support
        </b-navbar-item>
      </template>
    </b-navbar>
  </div>
</template>

<style>
.nav {
  background-color: #140928;
}

.container.nav {
  padding: 0 12px;
}
</style>
