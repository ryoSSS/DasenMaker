<template>
  <v-app>
    <v-app-bar color="primary" dense title absolute app>
      <!--
      <img src="/icon.png" style="max-height: 100%;" />
      -->
      <router-link tag="div" to="/">
        <v-toolbar-title class="toolbar-title">
          <span class="title-font">打線メーカー</span>
        </v-toolbar-title>
      </router-link>
      <v-spacer />

      <!--
      <template v-if="user.id === ''">
        <v-btn color="primary" depressed @click="login()">
          <v-icon size="25" left>mdi-twitter</v-icon>
          ログイン
        </v-btn>
      </template>

      <template v-else>
        <v-menu left bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-avatar size="40" v-bind="attrs" v-on="on" @click="() => {}">
              <img :src="user.photoURL" />
            </v-avatar>
          </template>
          <v-list>
            <v-list-item :to="'/user/' + user.id">
              <v-list-item-title>作成・参加したイベント</v-list-item-title>
            </v-list-item>
            <v-list-item @click="logout">
              <v-list-item-title>ログアウト</v-list-item-title>
            </v-list-item>
            <v-list-item @click="handleDeleteAccount">
              <v-list-item-title>退会する</v-list-item-title>
            </v-list-item>
          </v-list>
        </v-menu>
      </template>
      ーー>

      <!-
      <v-btn text rounded>カテゴリ</v-btn>
      <v-text-field dense single-line outlined label="キーワードを入力" prepend-inner-icon="mdi-magnify"></v-text-field>
      <v-spacer></v-spacer>
      -->
    </v-app-bar>

    <v-main class="content">
      <v-container
        style="
          margin-bottom: 100px;
          padding-left: 0;
          padding-right: 0;
          width: 100%;
        "
        fluid
      >
        <v-row
          style="margin: 0; padding: 0; width: 100%"
          align="center"
          justify="center"
        >
          <v-col style="padding: 0" cols="11" sm="6" md="4">
            <nuxt />
          </v-col>
        </v-row>
      </v-container>
    </v-main>

    <v-footer absolute app>
      <div>
        <!--
        <nuxt-link to="/contact">お問い合わせ</nuxt-link>
        <br />
        <nuxt-link to="/privacy_policy">プライバシーポリシー</nuxt-link>
        <br />
        <nuxt-link to="/terms">利用規約</nuxt-link>
        <br />
         -->
        <span>&copy;Copyright 2020 打線メーカー .</span>
      </div>
    </v-footer>
  </v-app>
</template>

<script lang="ts">
import { Vue, Component } from 'nuxt-property-decorator'

@Component
export default class Default extends Vue {}
</script>

<style lang="scss" scoped>
.title-font {
  color: white;
  cursor: pointer;
  font-size: 1.5rem;
  font-weight: 800;
}

.content {
  background-color: #f7f9fd;
}

.btn-title {
  color: white;
  font-size: 1rem;
  font-weight: bold;
}
</style>
