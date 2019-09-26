<template>
  <div id="reader">
    <p class="error">{{ error }}</p>
    <h1 class="exo">
      <span>{{ result }}</span>
    </h1>
    <h2 class="exo">{{ response }}</h2>
    <br />
    <h3>QRコードをかざしてください</h3>
    <qrcode-stream @decode="onDecode" @init="onInit" :id="status" />
    <br />
    <div id="btns">
      <v-btn color="primary" dark @click="clearAll">Clear</v-btn>
      <div>
        <v-row justify="center">
          <v-dialog v-model="dialog" persistent max-width="600px">
            <template v-slot:activator="{ on }">
              <v-btn color="primary" dark v-on="on">パスコード</v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="headline">パスコード入力</span>
              </v-card-title>
              <v-card-text>
                <p>パスコードは管理者に聞いてください</p>
                <v-container>
                  <v-row>
                    <v-col cols="12">
                      <v-text-field v-model="inputKey" label="Passcode" type="password" required></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
              <v-card-actions>
                <div class="flex-grow-1"></div>
                <v-btn color="blue darken-1" text @click="dialog = false">保存</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-row>
      </div>
      <!--Safari用-->
      <v-btn @click="playSound('/success.mp3')">Play</v-btn>
    </div>
    <p>※同じ名前のQR-Codeは連続で読み込めないので、別のPCで試してください。</p>
    <p>このPCでの受付履歴です。</p>
    <v-simple-table>
      <template v-slot:default>
        <thead>
          <tr>
            <th class="text-left">Time</th>
            <th class="text-left">AgentName</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in agentList" :key="item.index">
            <td class="text-left">{{ item.time }}</td>
            <td class="text-left">{{ item.name }}</td>
          </tr>
        </tbody>
      </template>
    </v-simple-table>
  </div>
</template>

<script>
import axios from "@nuxtjs/axios";

export default {
  layout: "client/simple",
  data() {
    return {
      result: "",
      error: "",
      response: "",
      inputKey: "",
      dialog: false,
      agentList: [],
      status: "",
      timer: null
    };
  },
  methods: {
    onDecode(result) {
      if (result != "") {
        this.result = result;
        if (this.timer) {
          clearTimeout(this.timer);
        }
        this.response = "検索中...";
        //Apps Scriptの公開URLにパラメータをつけてGETリクエスト
        this.$axios
          .$get(
            "Apps Scriptの公開URL",
            {
              params: {
                name: result,
                key: this.inputKey
              }
            }
          )
          .then(response => {
            if (response.length < 6) {
              this.response = "登録完了しました！ 受付時間： " + response;
              this.status = "ok-res";
              this.playSound("/success.mp3");
            } else {
              this.response = "ERROR : " + response;
              this.status = "ng-res";
              this.playSound("/error.mp3");
            }
            const sKey = localStorage.length + 1;
            let dataObj = { time: response, name: result };
            localStorage.setItem(sKey, JSON.stringify(dataObj));
            this.agentList.push(dataObj);
            this.timer = setTimeout(this.changeStatus, 6000);
            //console.log("response:" + response);
            result = "";
          })
          .catch(err => {
            this.response = err;
            this.status = "ng-res";
            console.log("error:" + err);
          });
      }
    },
    //vue-qrcode-readerのエラーログ
    async onInit(promise) {
      try {
        await promise;
      } catch (error) {
        if (error.name === "NotAllowedError") {
          this.error = "ERROR: you need to grant camera access permisson";
        } else if (error.name === "NotFoundError") {
          this.error = "ERROR: no camera on this device";
        } else if (error.name === "NotSupportedError") {
          this.error = "ERROR: secure context required (HTTPS, localhost)";
        } else if (error.name === "NotReadableError") {
          this.error = "ERROR: is the camera already in use?";
        } else if (error.name === "OverconstrainedError") {
          this.error = "ERROR: installed cameras are not suitable";
        } else if (error.name === "StreamApiNotSupportedError") {
          this.error = "ERROR: Stream API is not supported in this browser";
        }
      }
    },
    clearAll() {
      this.response = "";
      this.result = "";
      this.status = "";
    },
    changeStatus() {
      this.clearAll();
    },
    playSound(sound) {
      if (sound) {
        var audio = new Audio(sound);
        audio.play();
      }
    }
  },
  mounted() {
    console.log("reading from localstorage.");
    for (let i = 1; i <= localStorage.length; i++) {
      this.agentList.push(JSON.parse(localStorage.getItem(i)));
    }
  }
};
</script>

<style scoped>
#reader {
  text-align: center;
}
.exo {
  font-family: "Exo", sans-serif;
  text-transform: uppercase;
}
h1 {
  font-size: 3em;
}
#btns {
  display: flex;
  justify-content: center;
}
.v-btn {
  margin: 0 2em 2em 0;
}
#ok-res {
  border: solid 30px greenyellow;
}
#ng-res {
  border: solid 30px red;
}
</style>