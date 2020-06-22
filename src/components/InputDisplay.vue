<template>
  <v-content class="content">
    <v-container class="pt-5s">
      <div class="d-flex mb-1">
        <v-btn class="mr-5" small @click="switchTranslateMode">
          {{ inputLanguage }}
          <v-icon class="mx-2">mdi-swap-horizontal</v-icon>
          {{ outputedText }}
        </v-btn>
        <v-btn color="error" small @click="deleteText">
          <v-icon class="mr-2">mdi-delete</v-icon>DELETE
        </v-btn>
        <v-spacer></v-spacer>
        <v-dialog v-model="usageDialog" width="900">
          <template v-slot:activator="{ on, attrs }">
            <v-btn class="mr-5" small v-bind="attrs" v-on="on">How To Use</v-btn>
          </template>
          <v-card>
            <v-card-title class="headline grey lighten-2" primary-title>How To Use Morse Code Editor</v-card-title>
            <v-card-title>Morse Code Editorについて</v-card-title>
            <v-card-text>Morse Code Editorはモールス符号⇔英単語の翻訳ができるエディタです！翻訳はリアルタイムでプレビューすることができます。</v-card-text>
            <v-card-title>翻訳モードの切替え</v-card-title>
            <v-card-text>
              <v-btn class="mr-2" x-small>
                MORSE CODE
                <v-icon class="mx-2 mdi-18px">mdi-swap-horizontal</v-icon>ENGLISH
              </v-btn>をクリックすることで、翻訳モードを切り替えます。
            </v-card-text>
            <v-card-title>モールス符号入力モードの切替え</v-card-title>
            <v-card-text>
              モールス符号の入力にはキーボードの「.」と「-」を使用する『キーボードモード』とスペースキーを電鍵に見立てて使用し符号を入力する『電鍵モード』の2種類があります。
              <v-btn class="mr-2" x-small>
                <v-icon class="mdi-18px">mdi-keyboard</v-icon>
              </v-btn>をクリックすることでキーボードモードに、
              <v-btn class="mr-2" x-small>
                <v-icon class="mdi-18px">mdi-gesture-double-tap</v-icon>
              </v-btn>をクリックすることで電鍵モードに切り替えます。
            </v-card-text>
            <v-card-title class="subtitle-2">キーボードモード</v-card-title>
            <v-card-text>
              キーボードを使用してモールス符号を入力するモードです。
              <br />・ キーボードの「.」と「-」を使って入力してください。
              <br />・ 符号間は「 」(スペース)で、単語間は「 / 」（スペース+スラッシュ+スペース）で区切ってください。
              <br />・ バックスペースキーでひとつ前の符号やスペースを削除できます。
            </v-card-text>
            <v-card-title class="subtitle-2">電鍵モード</v-card-title>
            <v-card-text>
              スペースキーを電鍵に見立ててモールス符号を入力するモードです。
              <br />・ スペースキーを押してすぐ離すと短点「.」、スペースキーを少し長押ししてから離すと長点「-」を入力します。
              <br />・ スペースキーを離してから入力しない時間がしばらく続くと、1文字分の符号の入力が終了したとみなされ、「 」（スペース）が末尾に追加されます。
              <br />・ さらに入力しない時間が続くと、1単語分の符号の入力が終了されたとみなされ、半角スペースに続き「/ 」（スラッシュ+スペース）が末尾に追加されます。
              <br />・ バックスペースキーでひとつ前の符号やスペースを削除できます。
            </v-card-text>

            <v-divider></v-divider>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="primary" text @click="usageDialog = false">OK</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-btn
          class="mr-5"
          small
          @click="switchInputModeToKeyboard"
          :disabled="this.inputLanguage !== 'Morse Code' || this.inputMode === 'keyboard'"
        >
          <v-icon>mdi-keyboard</v-icon>
        </v-btn>
        <v-btn
          small
          @click="switchInputModeToSpaceKey"
          :disabled="this.inputLanguage !== 'Morse Code' || this.inputMode === 'telegraph'"
        >
          <v-icon>mdi-gesture-double-tap</v-icon>
        </v-btn>
      </div>
      <v-row class="textarea-container">
        <v-col cols="12" xs="12" md="6">
          <!-- モールス符号keyboard入力モード用のテキストエリア -->
          <v-textarea
            v-if="this.inputLanguage === 'Morse Code' && this.inputMode === 'keyboard'"
            :value="morseText"
            @input="inputText"
            :placeholder="keyboardModePlaceholder"
            no-resize
            outlined
            height="100%"
          ></v-textarea>
          <!-- モールス符号telegraph入力モード用のテキストエリア -->
          <v-textarea
            v-if="this.inputLanguage === 'Morse Code' && this.inputMode === 'telegraph'"
            v-model="morseText"
            @keydown.prevent.space.exact="keydownSpaceKey"
            @keyup.prevent.space.exact="keyupSpaceKey"
            :placeholder="telegraghModePlaceholder"
            no-resize
            outlined
            height="100%"
          ></v-textarea>
          <!-- 英文入力用のテキストエリア -->
          <v-textarea
            v-if="this.inputLanguage === 'English'"
            :value="englishText"
            @input="inputText"
            :placeholder="englishModePlaceholder"
            no-resize
            outlined
            height="100%"
          ></v-textarea>
        </v-col>
        <v-col class="compiledTextarea" cols="12" xs="12" md="6">
          <div
            v-if="this.inputLanguage === 'Morse Code'"
            class="markdown-body"
            v-html="outputedEnglishText"
          ></div>
          <div
            v-if="this.inputLanguage === 'English'"
            class="markdown-body"
            v-html="outputedMorseText"
          ></div>
        </v-col>
      </v-row>
    </v-container>
  </v-content>
</template>

<script lang="ts">
import Component from "vue-class-component";
import { Vue } from "vue-property-decorator";

@Component
export default class InputDisplay extends Vue {
  usageDialog = false;
  keyDownTime: number | undefined; // スペースキーを押したときのタイムスタンプ値
  keyUpTime: number | undefined; // スペースキーを離したときのタイムスタンプ値
  keyDownDuration: number | undefined; // スペースキーを押してから離すまでの時間（ミリ秒）
  dotDuration = 70; //dotの長さ（ミリ秒）
  dashDuration = this.dotDuration * 3; //dashの長さ（ミリ秒）。keyDownDurationがこの値未満ならドットを、以上ならダッシュを出力する
  characterDuration = this.dotDuration * 7; //文字の入力間隔（ミリ秒）
  wordDuration = this.dotDuration * 14; //単語の入力間隔（ミリ秒）
  characterDurationTimer: number | undefined;
  wordDurationTimer: number | undefined;
  morseText = "";
  englishText = "";
  inputLanguage: "Morse Code" | "English" = "Morse Code"; // 入力する言語
  outputedText: "Morse Code" | "English" = "English"; // 翻訳後に出力される言語
  inputMode: "keyboard" | "telegraph" = "telegraph"; // モールス符号の入力モード。

  keyboardModePlaceholder =
    "キーボードを使用してモールス符号を入力するモードです。\n\nこのテキストエリアをクリックして半角モードで入力してください。";
  telegraghModePlaceholder =
    "スペースキーを電鍵に見立ててモールス符号を入力するモードです。\n\nこのテキストエリアをクリックして半角モードで入力してください。";
  englishModePlaceholder =
    "英文をモールス信号に翻訳するモードです。\n\nこのテキストエリアに半角で入力してください。";

  // モールス符号→英数字の対応表
  morsePatternMap: { [s: string]: string } = {
    ".-": "A",
    "-...": "B",
    "-.-.": "C",
    "-..": "D",
    ".": "E",
    "..-.": "F",
    "--.": "G",
    "....": "H",
    "..": "I",
    ".---": "J",
    "-.-": "K",
    ".-..": "L",
    "--": "M",
    "-.": "N",
    "---": "O",
    ".--.": "P",
    "--.-": "Q",
    ".-.": "R",
    "...": "S",
    "-": "T",
    "..-": "U",
    "...-": "V",
    ".--": "W",
    "-..-": "X",
    "-.--": "Y",
    "--..": "Z",
    "-----": "0",
    ".----": "1",
    "..---": "2",
    "...--": "3",
    "....-": "4",
    ".....": "5",
    "-....": "6",
    "--...": "7",
    "---..": "8",
    "----.": "9",
    ".-.-.-": ".",
    "--..--": ",",
    "..--..": "?",
    ".----.": "'",
    "-....-": "-",
    "/": " " // スラッシュは単語間の区切りを示す
  };

  // 英数字の対応表→モールス符号の対応表
  morsePatternMapReverse: { [s: string]: string } = {};

  mounted(): void {
    // morsePatternMapのキーと値を反転させたmorsePatternMapReverseを生成する
    const patternMapKey: string[] = Object.keys(this.morsePatternMap);
    const patternMapValues: string[] = Object.values(this.morsePatternMap);
    for (let i = 0; i < patternMapKey.length; i++) {
      this.morsePatternMapReverse[patternMapValues[i]] = patternMapKey[i];
    }
  }

  // 入力される言語の切替え
  switchTranslateMode(): void {
    if (this.inputLanguage === "Morse Code") {
      this.inputLanguage = "English";
      this.outputedText = "Morse Code";
    } else {
      this.inputLanguage = "Morse Code";
      this.outputedText = "English";
    }
  }

  // テキストエリアをクリアする
  deleteText(): void {
    this.morseText = "";
    this.englishText = "";
  }

  // モールス符号の入力モードをキーボードモードに変更
  switchInputModeToKeyboard(): void {
    this.inputMode = "keyboard";
  }

  // モールス符号の入力モードを電鍵モードに変更
  switchInputModeToSpaceKey(): void {
    this.inputMode = "telegraph";
  }

  inputText(input: string): void {
    if (this.inputLanguage === "English") {
      this.englishText = input.toUpperCase();
    }
    if (this.inputMode === "keyboard") {
      this.morseText = input;
    }
  }

  keydownSpaceKey(): void {
    this.keyDownTime = Date.now();

    // characterDurationTimerとwordDurationTimerをリセットする
    if (typeof this.characterDurationTimer !== "undefined") {
      clearTimeout(this.characterDurationTimer);
    }
    if (typeof this.wordDurationTimer !== "undefined") {
      clearTimeout(this.wordDurationTimer);
    }
  }

  keyupSpaceKey(): void {
    this.keyUpTime = Date.now();

    if (
      typeof this.keyUpTime !== "undefined" &&
      typeof this.keyDownTime !== "undefined"
    ) {
      this.keyDownDuration = this.keyUpTime - this.keyDownTime;
    }

    if (
      typeof this.keyDownDuration !== "undefined" &&
      this.keyDownDuration < this.dashDuration
    ) {
      // スペースキーが押下されている時間(keyDownDuration)がdashDuration未満の場合、ドットを出力
      this.morseText += ".";
    } else {
      // dashDuration以上の場合、ダッシュを出力
      this.morseText += "-";
    }

    /* 前の入力から次の入力までの間隔がcharacterDurationより長い場合、
    一文字の入力が終了したとみなし、inputの末尾にスペースを追加する*/
    this.characterDurationTimer = setTimeout(() => {
      this.morseText += " ";
    }, this.characterDuration);

    /* 前の入力から次の入力までの間隔がwordDurationより長い場合、
    一単語の入力が終了したとみなし、inputの末尾に上記のスペースに加えスラッシュとスペースを追加する*/
    this.wordDurationTimer = setTimeout(() => {
      this.morseText += "/ ";
      const inputArray = this.morseText.split(" ");

      for (let i = 0; i < inputArray.length; i++) {
        if (Object.keys(this.morsePatternMap).includes(inputArray[i])) {
          inputArray[i] = this.morsePatternMap[inputArray[i]];
        }
      }
    }, this.wordDuration);
  }

  // モールス符号が入力されたとき、morsePatternMapと照らし合わせて対応する英数字を返す
  get outputedEnglishText(): string {
    const morseTextArray: string[] = this.morseText.split(" ");
    for (let i = 0; i < morseTextArray.length; i++) {
      if (Object.keys(this.morsePatternMap).includes(morseTextArray[i])) {
        morseTextArray[i] = this.morsePatternMap[morseTextArray[i]];
      }
    }

    return morseTextArray.join("");
  }

  // 英数字が入力されたとき、morsePatternMapReverseと照らし合わせて対応するモールス符号を返す
  get outputedMorseText(): string {
    let englishWordArray: string[] = this.englishText.split(" ");
    englishWordArray = englishWordArray.filter(value => value !== "");
    const joinedMorseWordArray: string[] = [];

    for (let i = 0; i < englishWordArray.length; i++) {
      const splitedMorseWordArray: string[] = [];
      const splitedEnglishWord: string[] = englishWordArray[i].split("");

      for (let j = 0; j < splitedEnglishWord.length; j++) {
        if (
          Object.keys(this.morsePatternMapReverse).includes(
            splitedEnglishWord[j]
          )
        ) {
          splitedMorseWordArray[j] = this.morsePatternMapReverse[
            splitedEnglishWord[j]
          ];
        }
      }
      joinedMorseWordArray[i] = splitedMorseWordArray.join(" ");
    }

    return joinedMorseWordArray.join(" / ");
  }
}
</script>

<style lang="scss">
.content {
  background-color: #eeeeee !important;
}
.v-main,
.v-main__wrap,
.container,
.v-textarea {
  height: 100%;
}
.button-container {
  height: 10%;
}
.textarea-container {
  height: 90%;
}
.v-input__control {
  height: 100% !important;
}
textarea {
  font-size: 1.5rem !important;

  &::placeholder {
    font-size: 1rem;
  }
}
.markdown-body {
  height: 100%;
  max-height: 70vh;
  overflow-y: scroll;
  font-size: 1.5rem;
}
.v-application code {
  background-color: #f6f8fa !important;
  color: unset !important;
}
</style>