<template>
    <div class="container">

        <div class="messages">
    <div class="message info" v-if="countdownSeconds > 0">
      Если не будет первой загрузки, токен будет удалён через: {{ countdownSeconds }} секунд
    </div>
    <div v-if="uploadMessage" :class="['message', messageClass]" v-html="uploadMessage"></div>
    <div v-if="message" :class="['message', messageClass]" v-html="message"></div>
  </div>

      <div v-if="isLoggedIn">
        <div class="avatar">
          <img :src="avatar" alt="Avatar" width="50px" />
          <span v-if="token" class="token">
            <template v-if="tokenShow">
              Твой токен: <strong>{{ token }}</strong>
            </template>
            <template v-else>
              Не забудь его сохранить, чтобы вспомнить!
            </template>
          </span>
      <button class="logout material-symbols-outlined" @click="logout"> exit_to_app </button>

        </div>
        <div class="surface">
          <div class="drop-zone" @click="openFileDialog" @dragover.prevent @dragenter="handleDragEnter" @dragleave="handleDragLeave" @drop.prevent="handleDrop">
            <button class="material-symbols-outlined wb">download</button>
            <span>Перетащите изображения сюда или нажмите для выбора файлов</span>
            <span style="opacity: 0.6;">Масимальный размер: 32MB</span>
          </div>
          <div v-if="uploads.length" class="upload-progress">
            <div class="upload-item" v-for="(item, index) in uploads" :key="index">
              <div class="type">{{ formatFileType(item.fileType) }}</div>
              <div class="name">{{ item.fileName }}</div>
              <div class="size">{{ formatFileSize(item.fileSize) }}</div>
              <div class="progress-bar">
                <div class="progress-fill" :style="{ width: item.progress + '%' }"></div>
              </div>
            </div>
          </div>
          <div v-if="uploadMessages.length" class="list">
            <div class="lupd" v-for="(msg, index) in uploadMessages" :key="index">
              <span class="material-symbols-outlined">check</span> {{ msg }}
            </div>
          </div>
          <div v-if="!album.length" class="empty-state">
            <div class="empty">
              <div class="empty-img"></div>
              <div class="empty-link">
                <div class="em-link"></div>
                <div class="em-copy"></div>
              </div>
            </div>
          </div>
          <div v-if="album.length" class="album">
            <div class="album-item" v-for="(img, index) in album" :key="img.firebaseKey">
              <img :src="img.url_medium" alt="загруженная пикча" />
              <div class="url">{{ img.url }}</div>
              <button class="material-symbols-outlined copy" @click="copyUrl(img)">
      <span v-if="img.copied">check</span>
      <span v-else>content_copy</span>
    </button>

            </div>
          </div>
        </div>
      </div>
      <div v-else class="surface">
        <div v-if="tokenGenerated">
          Твой токен: <strong>{{ token }}</strong>
          <br>
          Не забудь его сохранить, он как <b>пароль</b> и <b>логин</b>.
        </div>
        <div class="login">
          <input v-model="inputToken" placeholder="Вставь токен сюда" />
          <button class="nb" @click="login">Войти</button>
        </div>
        <button v-if="!tokenGenerated" class="db" @click="generateToken($event)" style="place-self: flex-start; display: flex; align-items: center; gap: 5px;">
          <span> Получить токен </span>
          <span class="material-symbols-outlined"> token </span>
        </button>
        <hr>
        <div class="faq">
          <div>
            <button class="spoiler" @click="toggleSpoiler('tokenInfo')" :class="{ open: spoilerStates.tokenInfo }">
              <span v-if="spoilerStates.tokenInfo" class="material-symbols-outlined"> keyboard_arrow_up </span>
              <span v-else class="material-symbols-outlined"> keyboard_arrow_down </span> Для чего нужен токен?
            </button>
            <div v-if="spoilerStates.tokenInfo" class="hdc">
              <span>Токен используется для быстрой авторизации, чтобы не заполнять формы регистрации. <br> То есть регистрация за один клик.</span>
            </div>
          </div>
          <div>
            <button class="spoiler" @click="toggleSpoiler('accountWhy')" :class="{ open: spoilerStates.accountWhy }">
              <span v-if="spoilerStates.accountWhy" class="material-symbols-outlined"> keyboard_arrow_up </span>
              <span v-else class="material-symbols-outlined"> keyboard_arrow_down </span> Зачем аккаунт?
            </button>
            <div v-if="spoilerStates.accountWhy" class="hdc">
              <span>Аккаунт позволяет сохранять историю загрузок, то есть чтобы не терять ссылки.</span>
            </div>
          </div>
          <div>
            <button class="spoiler" @click="toggleSpoiler('comment')" :class="{ open: spoilerStates.comment }">
              <span v-if="spoilerStates.comment" class="material-symbols-outlined"> keyboard_arrow_up </span>
              <span v-else class="material-symbols-outlined"> keyboard_arrow_down </span> Что сделать если не сохранил токен а там ссылки на изображения?
            </button>
            <div v-if="spoilerStates.comment" class="hdc">
              <span>Можешь новый токен получить, потерянное не вернешь.</span>
            </div>
          </div>
          <div>
            <button class="spoiler" @click="toggleSpoiler('delete')" :class="{ open: spoilerStates.delete }">
              <span v-if="spoilerStates.delete" class="material-symbols-outlined"> keyboard_arrow_up </span>
              <span v-else class="material-symbols-outlined"> keyboard_arrow_down </span> Как удалить изображение?
            </button>
            <div v-if="spoilerStates.delete" class="hdc">
              <span>Отправь <span class="feedback" @click="toggleSpoiler('feedback')" :class="{ open: spoilerStates.feedback }">
                  <span style="color: var(--dot); cursor: pointer;">сюда</span>
                </span> свой токен и ссылку на изображение</span>
              <br><br>
              <form class="form" action="https://formspree.io/f/mnnjvbre" method="POST">
                <input type="text" name="text" placeholder="твой токен" required>
                <input type="text" name="text" placeholder="ссылка которую нужно удалить" required>
                <button class="nb" type="submit">отправить</button>
              </form>
            </div>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import useAppLogic from "./set.js";
  export default {
    name: "App",
    setup() {
      return useAppLogic();
    }
  };
  </script>
  