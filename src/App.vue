<template>
  <div id="app">
    <header class="header">
      <h1>Chat</h1>
      <!-- ログイン時にはフォームとログアウトボタンを表示 -->
      <div v-if="user.uid" key="login">
        [{{ user.displayName }}]
        <button type="button" @click="doLogout">ログアウト</button>
      </div>
      <!-- 未ログイン時にはログインボタンを表示 -->
      <div v-else key="logout">
        <button type="button" @click="doLogin">ログイン</button>
      </div>
    </header>

    <!--Firebaseから取得したリストを描画（トランジション付き）-->
    <transition-group name="chat" tag="div" class="list content">
      <section v-for="{ postId, name, image, message } in chat" :key="postId" class="item">
        <div class="item-image"><img :src="image" width="40" height="40"></div>
        <div class="item-detail">
          <div class="item-name">{{ name }}</div>
          <div class="item-message" :text="message">
            {{ message }}
          </div>
        </div>
      </section>
    </transition-group>
  
    <!-- 入力フォーム -->
    <form action="" @submit.prevent="doSend" class="form">
      <textarea
        v-model="input"
        :disabled="!user.uid"
        @keydown.enter.exact.prevent="doSend"></textarea>
      <button type="submit" :disabled="!user.uid" class="send-button">Send</button>
    </form>
  </div>
</template>

<script>
// firebase モジュール
import firebase from 'firebase/app'
import 'firebase/firestore';
import 'firebase/auth';
const config = {
  apiKey: "AIzaSyAZRyPO91_W3bL6vXXbpa6dzsH8fd_Vuo4",
  authDomain: "vue-chat-96ccf.firebaseapp.com",
  projectId: "vue-chat-96ccf",
  databaseURL: "https://vue-chat-96ccf.firebaseio.com",
  storageBucket: "vue-chat-96ccf.appspot.com",
  messagingSenderId: "361824680795",
  appId: "1:361824680795:web:82c0c8e5932ce80976b87e",
  measurementId: "G-JX3QYKZ3C2",
};
firebase.initializeApp(config);
const db = firebase.firestore();
export default {
  components: {  },
  data() {
    return {
      user: {},  // ユーザー情報
      chat: [],  // 取得したメッセージを入れる配列
      input: ''  // 入力したメッセージ
    }
  },
  created() {
    firebase.auth().onAuthStateChanged(user => {
      this.user = user ? user : {}
      db.collection('messages').orderBy('createdAt', 'asc').onSnapshot((snapshot) => {
        const messages = snapshot.docs.map((doc) => doc.data());
        
      if (user) {
        this.chat = []
        this.chat = messages
        this.scrollBottom()
      } else {
        this.chat = []
      }
      })
      
    })
  },
    methods: {
    // ログイン処理
    doLogin() {
      const provider = new firebase.auth.GoogleAuthProvider()
      firebase.auth().signInWithPopup(provider)
    },
    // ログアウト処理
    doLogout() {
      firebase.auth().signOut()
    },
    // スクロール位置を一番下に移動
    scrollBottom() {
      this.$nextTick(() => {
        window.scrollTo(0, document.body.clientHeight)
      })
    },
    doSend() {
      if (this.user.uid && this.input.length) {
        const S =
        "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
        const N = 16;
        const randomChar = Array.from(crypto.getRandomValues(new Uint32Array(N)))
        .map((n) => S[n % S.length])
        .join("");
        db.collection('messages').add({
          authorId: this.user.uid,
          postId: randomChar,
          message: this.input,
          name: this.user.displayName,
          image: this.user.photoURL,
          createdAt: firebase.firestore.Timestamp.now(),
        }).then((doc) => {
          doc.onSnapshot()
          console.log()
          this.input = ''
        })
      }
    }
  }
}
</script>

<style>
* {
  margin: 0;
  box-sizing: border-box;
}
.header {
  background: #3ab383;
  margin-bottom: 1em;
  padding: 0.4em 0.8em;
  color: #fff;
}
.content {
  margin: 0 auto;
  padding: 0 10px;
  max-width: 600px;
}
.form {
  position: fixed;
  display: flex;
  justify-content: center;
  align-items: center;
  bottom: 0;
  height: 80px;
  width: 100%;
  background: #f5f5f5;
}
.form textarea {
  border: 1px solid #ccc;
  border-radius: 2px;
  height: 4em;
  width: calc(100% - 6em);
  resize: none;
}
.list {
  margin-bottom: 100px;
}
.item {
  position: relative;
  display: flex;
  align-items: flex-end;
  margin-bottom: 0.8em;
}
.item-image img {
  border-radius: 20px;
  vertical-align: top;
}
.item-detail {
  margin: 0 0 0 1.4em;
}
.item-name {
  font-size: 75%;
}
.item-message {
  position: relative;
  display: inline-block;
  padding: 0.8em;
  background: #deefe8;
  border-radius: 4px;
  line-height: 1.2em;
}
.item-message::before {
  position: absolute;
  content: " ";
  display: block;
  left: -16px;
  bottom: 12px;
  border: 4px solid transparent;
  border-right: 12px solid #deefe8;
}
.send-button {
  height: 4em;
}
/* トランジション用スタイル */
.chat-enter-active {
  transition: all 1s;
}
.chat-enter {
  opacity: 0;
  transform: translateX(-1em);
}
</style>
