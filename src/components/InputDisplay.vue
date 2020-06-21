<template>
  <v-content class="content">
    <v-container class="pt-5s">
      <div class="d-flex mb-1">
        <v-btn class="mr-5" small @click="switchTranslateMode">
          {{ originalLanguage }}
          <v-icon class="mx-2">mdi-ray-start-arrow</v-icon>
          {{ compiledLanguage }}
        </v-btn>
        <v-btn color="error" small @click="deleteText">
          <v-icon class="mr-2">mdi-delete</v-icon>DELETE
        </v-btn>
        <v-spacer></v-spacer>
        <v-btn
          class="mr-5"
          small
          @click="switchInputModeToKeyboard"
          :disabled="this.originalLanguage !== 'Morse Code' || this.inputMode === 'keyboard'"
        >
          <v-icon class="mr-2">mdi-keyboard</v-icon>Keyboard Mode
        </v-btn>
        <v-btn
          small
          @click="switchInputModeToSpaceKey"
          :disabled="this.originalLanguage !== 'Morse Code' || this.inputMode === 'telegraph'"
        >
          <v-icon class="mr-2">mdi-gesture-double-tap</v-icon>Telegraph Key Mode
        </v-btn>
      </div>
      <v-row class="textarea-container">
        <v-col cols="12" xs="12" md="6">
          <v-textarea
            v-if="this.originalLanguage === 'Morse Code' && this.inputMode === 'keyboard'"
            :value="morseText"
            @input="inputText"
            :placeholder="keyboardModePlaceholder"
            no-resize
            outlined
            height="100%"
          ></v-textarea>
          <v-textarea
            v-if="this.originalLanguage === 'Morse Code' && this.inputMode === 'telegraph'"
            v-model="morseText"
            @keydown.prevent.space.exact="keydownSpaceKey"
            @keyup.prevent.space.exact="keyupSpaceKey"
            :placeholder="telegraghModePlaceholder"
            no-resize
            outlined
            height="100%"
          ></v-textarea>
          <v-textarea
            v-if="this.originalLanguage === 'English'"
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
            v-if="this.originalLanguage === 'Morse Code'"
            class="markdown-body"
            v-html="outputedEnglishText"
          ></div>
          <div
            v-if="this.originalLanguage === 'English'"
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
  originalLanguage: "Morse Code" | "English" = "Morse Code";
  compiledLanguage: "Morse Code" | "English" = "English";
  inputMode: "keyboard" | "telegraph" = "telegraph";

  keyboardModePlaceholder =
    "キーボードを使用してモールス符号を入力するモードです。\n\n・このテキストエリアをクリックして半角モードで入力してください。\n・キーボードの「.」と「-」を使って入力してください。\n・符号間は「 」(スペース)で、単語間は「 / 」（スペース+スラッシュ+スペース）で区切ってください。\n・バックスペースキーでひとつ前の符号やスペースを削除できます。";
  telegraghModePlaceholder =
    "スペースキーを電鍵に見立ててモールス符号を入力するモードです。\n\n・このテキストエリアをクリックして半角モードで入力してください。\n・スペースキーを押してすぐ離すと短点「.」、スペースキーを少し長押ししてから離すと長点「-」を入力できます。\n・スペースキーを離してから入力しない時間がしばらく続くと、1文字分の符号の入力が終了したとみなされ、「 」（スペース）が末尾に追加されます。\n・さらに入力しない時間が続くと、1単語分の符号の入力が終了されたとみなされ、半角スペースに続き「/ 」（スラッシュ+スペース）が末尾に追加されます。\n・バックスペースキーでひとつ前の符号やスペースを削除できます。";
  englishModePlaceholder =
    "英文をモールス信号に翻訳するモードです。\n\nこのテキストエリアに半角モードで入力してください。";

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

  morsePatternMapReverse: { [s: string]: string } = {};

  mounted(): void {
    const patternMapKey: string[] = Object.keys(this.morsePatternMap);
    const patternMapValues: string[] = Object.values(this.morsePatternMap);
    for (let i = 0; i < patternMapKey.length; i++) {
      this.morsePatternMapReverse[patternMapValues[i]] = patternMapKey[i];
    }
  }

  switchTranslateMode(): void {
    if (this.originalLanguage === "Morse Code") {
      this.originalLanguage = "English";
      this.compiledLanguage = "Morse Code";
    } else {
      this.originalLanguage = "Morse Code";
      this.compiledLanguage = "English";
    }
  }

  deleteText(): void {
    this.morseText = "";
    this.englishText = "";
  }

  switchInputModeToKeyboard(): void {
    this.inputMode = "keyboard";
  }

  switchInputModeToSpaceKey(): void {
    this.inputMode = "telegraph";
  }

  inputText(input: string): void {
    if (this.originalLanguage === "English") {
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
      // マウスボタンが押下されている時間がdashDuration未満の場合、ドットを出力
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
    一単語の入力が終了したとみなし、inputの末尾にスペースとスラッシュを追加する*/
    this.wordDurationTimer = setTimeout(() => {
      this.morseText += "/ ";
      const inputArray = this.morseText.split(" ");

      for (let i = 0; i < inputArray.length; i++) {
        if (Object.keys(this.morsePatternMap).includes(inputArray[i])) {
          inputArray[i] = this.morsePatternMap[inputArray[i]];
        }
      }
    }, this.wordDuration);

    this.keyDownTime = undefined;
  }

  get outputedEnglishText(): string {
    const morseTextArray: string[] = this.morseText.split(" ");
    for (let i = 0; i < morseTextArray.length; i++) {
      if (Object.keys(this.morsePatternMap).includes(morseTextArray[i])) {
        morseTextArray[i] = this.morsePatternMap[morseTextArray[i]];
      }
    }

    return morseTextArray.join("");
  }

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