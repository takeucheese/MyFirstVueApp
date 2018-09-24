<template>
  <div class="editor">
    <h1>エディター一覧</h1>
    <template v-if="messages">
      <div class="message" v-for="(message, index) in messages" :key="index">{{ message }}</div>
    </template>
    <span>{{ user.displayName }}</span>
    <button v-on:click="logout">ログアウト</button>
    <div class="memoListWrapper">
      <div
        class="memoList"
        v-for="(memo, index) in memos"
        @click="selectMemo(index)"
        :key="index"
        :data-selected="index == selectIndex">
        <p class="memoTitle">{{ memo.createTime }} {{ displayTitle(memo.markdown) }}</p>
      </div>
      <button id="addMemoBtn" @click="addMemo">メモの追加</button>
      <button id="deleteMemoBtn" v-if="memos.length > 1" @click="deleteMemo">選択中のメモの削除</button>
      <button id="saveMemosBtn" @click="saveMemos">メモの保存</button>
    </div>
    <textarea
        class="markdown"
        v-model="memos[selectIndex].markdown">
    </textarea>
    <div class="preview markdown-body" v-html="preview()"></div>
  </div>
</template>

<script>
import marked from 'marked'
import moment from 'moment'
export default {
  name: 'editor',
  props: ['user'],
  data () {
    return {
      messages: [],
      memos: [
        {
          markdown: 'no title',
          createTime: moment().format('YYYY-MM-DD hh:mm:ss')
        }
      ],
      garbages: [],
      selectIndex: 0
    }
  },
  mounted: function () {
    document.onkeydown = e => {
      if (e.key == 's' && e.metaKey) {
        this.saveMemos()
        return false
      }
    }
  },
  created: function () {
    const firestore = firebase.firestore();
    const settings = {
      /* your settings... */ 
      timestampsInSnapshots: true
    };
    firestore.settings(settings);
    firestore
      .collection('memos')
      .doc(this.user.uid)
      .get()
      .then(doc => {
        if (doc.exists && doc.data().memos) {
          this.memos = doc.data().memos
        }
      })
  },
  beforeDestroy: function () {
    document.onkeydown = null
  },
  methods: {
    logout: function () {
      firebase.auth().signOut()
    },
    deleteMessages: function () {
      this.messages = []
    },
    addMemo: function () {
      this.deleteMessages()
      this.memos.push({
        markdown: 'no title',
        createTime: moment().format('YYYY-MM-DD hh:mm:ss')
      })
      this.selectIndex = this.memos.length - 1
    },
    saveMemos: function () {
      this.deleteMessages()
      const firestore = firebase.firestore()
      const settings = {
        /* your settings... */ 
        timestampsInSnapshots: true
      }
      firestore
        .settings(settings)
      firestore
        .collection('memos')
        .doc(this.user.uid)
        .set({ memos: this.memos })
        .then(() => {
          //TODO: 失敗時もメッセージが出るようにしたい 
          this.messages.push('保存しました')
        })
    },
    deleteMemo: function () {
      this.deleteMessages()
      this.memos[this.selectIndex].deleteTime = moment().format('YYYY-MM-DD hh:mm:ss')
      this.garbages.push(this.memos[this.selectIndex])
      console.log(this.garbages)
      this.memos.splice(this.selectIndex, 1)
      if (this.selectIndex > 0) {
        this.selectIndex--
      }
    },
    selectMemo: function (index) {
      this.deleteMessages()
      this.selectIndex = index
    },
    preview: function () {
      return marked(this.memos[this.selectIndex].markdown)
    },
    displayTitle: function (text) {
      return text.split(/\n/)[0]
    }
  }
}
</script>

<style lang="scss" scoped>
.memoListWrapper {
  width: 19%;
  float: left;
  border-top: 1px solid #000;
}
.memoList {
  padding: 10px;
  box-sizing: border-box;
  text-align: left;
  border-bottom: 1px solid #000;
  &:nth-child(even) {
    background-color: #ccc;
  }
  &[data-selected="true"] {
      background-color: #ccf;
  }
}
.message {
  color: #22f;
}
.memoTitle {
  height: 1.5em;
  margin: 0;
  white-space: nowrap;
  overflow: hidden;
}
#addMemoBtn {
  margin-top: 20px;
}
#deleteMemoBtn {
  margin: 10px;
}
.markdown {
  float: left;
  width: 40%;
  height: 500px;
}
.preview {
  float: left;
  width: 40%;
  text-align: left;
}
</style>
