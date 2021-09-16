<template>
  <b-navbar>
    <template slot="brand">
      <b-navbar-item tag="router-link" :to="{ path: '/' }">
        <img src="@/assets/images/soomgo.png" alt="Soomgo" />
      </b-navbar-item>
    </template>
    <template slot="start">
      <b-navbar-item to="/about" tag="router-link">
        About
      </b-navbar-item>
    </template>

    <template slot="end"> </template>
  </b-navbar>
</template>
