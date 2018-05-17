<template>
						<form class="has-text-centered">
							<div class="box" dir="rtl">
								<div class="field">
									<label for="id" class="label is-medium has-text-right">رقم المتطوعة</label>
									<div class="control" :class="{'is-loading': isLoading}">
										<input name="id" type="tel" v-model="id" class="input is-primary is-large" 
                    :class="{'is-danger': errors.has('id'), 'is-danger': (res === false), 'is-success': (res === true)}" 
                    @input="inputHandler" data-vv-delay="1000" data-vv-as="رقم المتطوعة" v-validate="'required|numeric'">
									</div>
									<div class="help has-text-right is-size-6" 
                  :class="[res ? 'is-success' : 'is-danger', errors.has('id') ? 'is-danger': '']">
                  {{errors.has("id") ? errors.first("id"): helpText}}</div>
									<div v-if="!login && loginTime !== ''" class="help is-info is-size-6 is-2">وقت الدخول: {{loginTime}}<br>عدد الساعات: {{hoursSince}}</div>
								</div>
							</div>
							<button class="button is-large is-rounded" @click.prevent="submit" 
              :class="[login ? 'is-success' : 'is-danger']" type="submit" 
              :disabled="(true)">{{btnText}}</button>
						</form>
</template>
<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import ar from "vee-validate/dist/locale/ar";
import { Validator } from "vee-validate";

import debounce from "lodash/debounce";
import axios from "axios";

interface Info {
  id: number;
  name: string;
  vol_in: number | null;
  vol_out: number | null;
}

Validator.localize("ar", ar);

@Component
export default class Login extends Vue {
  id = "";

  //indicates login status
  isLoading = false;
  //default recevice values
  info: Info = { id: 0, name: "", vol_in: null, vol_out: null };
  res: boolean | null = null;
  helpText = "";
  get getURL() {
    return "get.php?id=" + this.id;
  }
  get postURL() {
    return "post.php?id=" + this.id;
  }

  //time
  get loginTime() {
    if (this.info.vol_in !== null) {
      const d = new Date(this.info.vol_in * 1000).toLocaleString("ar-SA");
      return d;
    }
    return "";
  }
  get hoursSince() {
    if (this.info.vol_in !== null) {
      const now = new Date().getTime();
      const time = (this.info.vol_in - 0) * 1000;
      const diff = (now - time) / (1000 * 3600); //milliseconds * seconds in an hour
      return Math.round(diff);
    }
    return "";
  }

  //button text
  get btnText() {
    return this.login ? "دخول" : "خروج";
  }

  //Button supposed action
  get login(): boolean {
    if (
      (this.info.vol_in === null && this.info.vol_out === null) ||
      (this.info.vol_in !== null && this.info.vol_out !== null)
    )
      return true;

    return false;
  }

  //OnInput
  inputHandler() {
    if (this.id === "") {
      this.$validator.reset();
      if (this.helpText !== "") this.resetHelp();
      return;
    }
    this.$validator.validateAll().then(res => {
      if (res === true) {
        this.isLoading = true;
        this.resetHelp();
        this.getInfo();
      }
    });
  }

  //OnSubmit
  submit() {
    this.$validator.validateAll().then(res => {
      if (res === true) {
        this.isLoading = true;
        this.resetHelp();
        this.postInfo();
      }
    });
  }

  //Get info based on input
  async getInfo() {
    try {
      const res = await axios.get(this.getURL);
      const obj: { status: boolean; data?: Info } = res.data;
      if (obj.status === true) {
        this.info = <Info>obj.data;
        this.helpText = this.info.name;
        this.res = true;
        this.isLoading = false;
      } else {
        this.helpText = "تأكدي أنكِ أدخلتِ رقمكِ بشكلٍ صحيح";
        this.res = false;
        this.isLoading = false;
      }
    } catch (e) {
      this.helpText = "حدث خطأ";
      this.res = false;
      this.isLoading = false;
    }
  }

  //send info to the server
  async postInfo() {
    try {
      const res = await axios.get(this.postURL);
      const obj: { status: boolean } = res.data;
      if (obj.status) {
        this.res = true;
        if (this.login) {
          this.helpText = `تم تسجيل دخول ${this.info.name} بنجاح.`;
        } else {
          this.helpText = `تم تسجيل خروج ${this.info.name} بنجاح`;
        }
        setTimeout(this.resetHelp, 3000);
      }
    } catch (error) {
      this.res = false;
      this.helpText = "حدث خطأ عند إرسال الطلب";
    }
  }

  resetHelp() {
    this.res = null;
    this.helpText = "";
  }

  mounted() {
    this.getInfo = debounce(this.getInfo, 500);
  }
}
</script>
