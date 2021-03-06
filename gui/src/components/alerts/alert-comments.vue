/*
* CORTX-CSM: CORTX Management web and CLI interface.
* Copyright (c) 2020 Seagate Technology LLC and/or its Affiliates
* This program is free software: you can redistribute it and/or modify
* it under the terms of the GNU Affero General Public License as published
* by the Free Software Foundation, either version 3 of the License, or
* (at your option) any later version.
* This program is distributed in the hope that it will be useful,
* but WITHOUT ANY WARRANTY; without even the implied warranty of
* MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
* GNU Affero General Public License for more details.
* You should have received a copy of the GNU Affero General Public License
* along with this program. If not, see <https://www.gnu.org/licenses/>.
* For any questions about this software or licensing,
* please email opensource@seagate.com or cortx-questions@seagate.com.
*/
<template>
  <div class="cortx-modal-container" v-if="value">
    <div class="cortx-modal">
      <div class="cortx-modal-header">
        <label>{{ $t("alerts.comments") }}</label>
        <img id="alert-closeadd-comment-dialog"
          class="cortx-modal-close"
          :src="require('@/assets/close-green.svg')"
          @click="closeAddCommentsDialog()"
        />
      </div>
      <div>
        <div class="cortx-comments-container-overlay" v-if="showLoader"></div>
        <div class="cortx-comments-container">
          <label v-if="alertComments.length === 0" class="cortx-text-md ml-4">{{ $t("alerts.noComments") }}</label>
          <div
            v-else
            class="cortx-comment"
            v-for="alertComment in alertComments"
            :key="alertComment.comment_id"
          >
            <div>
              <span class="cortx-text-md">{{ alertComment.comment_text }}</span>
            </div>
            <div class="mt-2" style="height: 20px;">
              <span class="cortx-text-sm">{{ new Date(alertComment.created_time * 1000) | timeago }}</span>
              <span class="cortx-text-sm mx-3">|</span>
              <span class="cortx-text-sm">{{ alertComment.created_by }}</span>
            </div>
          </div>
        </div>
         <cortx-has-access :to="$cortxUserPermissions.alerts + $cortxUserPermissions.update">
        <div class="cortx-modal-footer">
          <div
            class="cortx-form-group"
            :class="{
            'cortx-form-group--error': $v.addCommentForm.comment_text.$error
            }"
            style="width: 100%;"
          >
         
            <textarea
             id="alert-comment-textarea"
              class="cortx-form__input_textarea"
              v-model.trim="addCommentForm.comment_text"
              @input="$v.addCommentForm.comment_text.$touch"
            ></textarea>
           
            <div class="cortx-form-group-label cortx-form-group-error-msg">
              <label
                v-if="$v.addCommentForm.comment_text.$dirty && !$v.addCommentForm.comment_text.required"
              >{{ $t("alerts.commentRequired") }}</label>
              <label
                v-else-if="$v.addCommentForm.comment_text.$dirty && !$v.addCommentForm.comment_text.maxLength"
              >{{ $t("alerts.commentCannotBeMoreThan250Char") }}</label>
            </div>
          </div>
          <div style="height: 40px;margin-top:10px;">
            <button
             id="alert-close-comment-dialogbtn"
              type="button"
              class="cortx-btn-secondary cortx-float-r ml-3"
              @click="closeAddCommentsDialog()"
            >Cancel</button>
            <button
              id="alert-save-commnetbtn"
              type="button"
              class="cortx-btn-primary cortx-float-r"
              @click="saveComment()"
              :disabled="$v.addCommentForm.$invalid"
            >Save</button>
          </div>
        </div>
          </cortx-has-access>
      </div>
    </div>
  </div>
</template>
<script lang="ts">
import { Component, Vue, Prop, Watch } from "vue-property-decorator";
import { Validations } from "vuelidate-property-decorators";
import { required, maxLength } from "vuelidate/lib/validators";
import { Api } from "./../../services/api";
import apiRegister from "./../../services/api-register";
import { AlertComment } from "../../models/alert";
import i18n from "./alert.json";

@Component({
  name: "cortx-alert-comments",
  i18n: {
    messages: i18n
  }
})
export default class CortxAlertComments extends Vue {
  @Prop({ required: true })
  public value: boolean;
  @Prop({ required: true })
  public alertId: string;
  public alertComments: AlertComment[] = [];
  public showLoader: boolean = false;
  public addCommentForm = {
    comment_text: ""
  };

  @Validations()
  public validations = {
    addCommentForm: {
      comment_text: { required, maxLength: maxLength(250) }
    }
  };

  @Watch("value")
  public async onPropertyChanged(newValue: number, oldValue: number) {
    if (newValue) {
      await this.getAlertComments();
    }
  }

  public async getAlertComments() {
    this.showLoader = true;
    const res = await Api.getAll(
      apiRegister.all_alerts + "/" + this.alertId + "/comments"
    );
    if (res.data) {
      this.alertComments = res.data.comments;
    }
    this.showLoader = false;
  }

  public async saveComment() {
    this.showLoader = true;
    const res = await Api.post(
      apiRegister.all_alerts + "/" + this.alertId + "/comments",
      { comment_text: this.addCommentForm.comment_text }
    );
    this.resetAddCommentForm();
    this.showLoader = false;
    await this.getAlertComments();
  }

  public closeAddCommentsDialog() {
    this.alertComments = [];
    this.resetAddCommentForm();
    this.$emit("input", false);
  }

  private resetAddCommentForm() {
    this.addCommentForm.comment_text = "";
    if (this.$v.addCommentForm) {
      this.$v.addCommentForm.$reset();
    }
  }
}
</script>
<style lang="scss" scoped>
.cortx-comments-container-overlay {
  height: 24em;
  width: 424px;
  position: absolute;
  background-color: #ffffff;
  opacity: 70%;
}
.cortx-comments-container {
  height: 12.5em;
  border-bottom: 1px solid #b7b7b7;
  overflow: auto;
  padding: 2px;
}
.cortx-comment {
  padding: 0.5em;
  border: 1px solid #b7b7b7;
  margin-bottom: 2px;
  border-radius: 4px;
}
.cortx-modal-footer {
  height: 12em;
  padding: 0.5em;
}
</style>
