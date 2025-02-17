<template>
  <div>
    <Modal v-show="isNoIncomeAndFiles" @close="isNoIncomeAndFiles = false">
      <template v-slot:body>
        <div class="fr-container">
          <div class="fr-grid-row justify-content-center">
            <div class="fr-col-12">
              <p>
                {{
                  $t(
                    "guarantorfinancialdocumentform.warning-no-income-and-file"
                  )
                }}
              </p>
            </div>
          </div>
        </div>
      </template>
    </Modal>
    <ConfirmModal
      v-if="isDocDeleteVisible"
      @valid="validSelect()"
      @cancel="undoSelect()"
    >
      <span>{{
        $t(`explanation-text.${guarantorKey()}.will-delete-files`)
      }}</span>
    </ConfirmModal>
    <ValidationObserver v-slot="{ validate }">
      <NakedCard class="fr-p-md-5w fr-mb-3w">
        <h1 class="fr-h6">
          {{ $t("guarantorfinancialdocumentform.financial") }}
        </h1>
        <form name="form" @submit.prevent="validate().then(save())">
          <div>
            <div>
              <div>
                {{ $t("guarantorfinancialdocumentform.select-label") }}
              </div>

              <TroubleshootingModal>
                <GuarantorChoiceHelp></GuarantorChoiceHelp>
                <DocumentInsert
                  :allow-list="financialDocument.documentType.acceptedProofs"
                  :block-list="financialDocument.documentType.refusedProofs"
                  v-if="financialDocument.documentType.key"
                ></DocumentInsert>
              </TroubleshootingModal>

              <div class="fr-mt-3w">
                <fieldset class="fr-fieldset">
                  <div class="fr-fieldset__content">
                    <div class="fr-grid-row">
                      <div
                        v-for="d in documents"
                        :key="d.key"
                        class="full-width-xs"
                      >
                        <BigRadio
                          :val="d"
                          v-model="financialDocument.documentType"
                          @input="onSelectChange()"
                        >
                          <div class="fr-grid-col spa">
                            <span>{{ $t(`documents.${d.key}`) }}</span>
                          </div>
                        </BigRadio>
                      </div>
                    </div>
                  </div>
                </fieldset>
              </div>
            </div>
          </div>
          <div
            class="fr-mt-3w"
            v-if="
              financialDocument.documentType.key &&
                financialDocument.documentType.key
            "
          >
            <div
              v-if="
                financialDocument.documentType &&
                  financialDocument.documentType.key
              "
            >
              <div>
                <validation-provider
                  :rules="{ required: true, regex: /^[0-9 ]+$/ }"
                  v-slot="{ errors, valid }"
                >
                  <div
                    class="fr-input-group"
                    :class="{
                      'fr-input-group--error': errors[0]
                    }"
                  >
                    <label for="monthlySum" class="fr-label"
                      >{{ $t("guarantorfinancialdocumentform.monthlySum-label") }} :</label
                    >
                    <input
                      id="monthlySum"
                      :placeholder="$t('guarantorfinancialdocumentform.monthlySum')"
                      type="number"
                      min="0"
                      step="1"
                      v-model="financialDocument.monthlySum"
                      name="monthlySum"
                      class="validate-required form-control fr-input"
                      :class="{
                        'fr-input--valid': valid,
                        'fr-input--error': errors[0]
                      }"
                      required
                    />
                    <span class="fr-error-text" v-if="errors[0]">{{
                      $t(errors[0])
                    }}</span>
                    <span
                      class="fr-error-text"
                      v-if="financialDocument.monthlySum > 10000"
                    >
                      {{ $t("guarantorfinancialdocumentform.high-salary") }}
                    </span>
                    <span
                      class="fr-error-text"
                      v-if="
                        financialDocument.monthlySum !== '' &&
                          financialDocument.monthlySum <= 0
                      "
                    >
                      {{ $t("guarantorfinancialdocumentform.low-salary") }}
                    </span>
                  </div>
                </validation-provider>
              </div>
            </div>
          </div>
        </form>
        <div
          class="fr-mt-3w"
          v-if="
            financialDocument.documentType.key &&
              financialDocument.documentType.key !== 'no-income' &&
              financialDocument.monthlySum >= 0 &&
              financialDocument.monthlySum !== ''
          "
        >
          <div class="fr-mb-3w">
            <p
              v-html="
                $t(
                  `explanation-text.${guarantorKey()}.${
                    financialDocument.documentType.key
                  }`
                )
              "
            ></p>
          </div>
          <AllDeclinedMessages
            class="fr-mb-3w"
            :documentDeniedReasons="documentDeniedReasons"
            :documentStatus="documentStatus"
          ></AllDeclinedMessages>
          <div v-if="financialFiles().length > 0" class="fr-col-md-12 fr-mb-3w">
            <ListItem
              v-for="(file, k) in financialFiles()"
              :key="k"
              :file="file"
              @remove="remove(financialDocument, file)"
            />
          </div>
          <div class="fr-mb-3w">
            <FileUpload
              :current-status="financialDocument.fileUploadStatus"
              @add-files="addFiles(financialDocument, ...arguments)"
              @reset-files="resetFiles(financialDocument, ...arguments)"
            ></FileUpload>
          </div>
          <div class="fr-col-12 fr-mb-3w bg-purple fr-checkbox-group">
            <input
              type="checkbox"
              id="noDocument"
              value="false"
              v-model="financialDocument.noDocument"
            />
            <label for="noDocument">
              {{
                $t(
                  `explanation-text.${guarantorKey()}.${getCheckboxLabel(
                    financialDocument.documentType.key
                  )}`
                )
              }}
            </label>
          </div>
          <div class="fr-mb-5w" v-if="financialDocument.noDocument">
            <validation-provider
              :rules="{ required: true }"
              v-slot="{ errors, valid }"
            >
              <div class="fr-input-group">
                <label class="fr-label" for="customText">
                  {{
                    $t(
                      `explanation-text.${guarantorKey()}.${getCustomTextLabel(
                        financialDocument.documentType.key
                      )}`
                    )
                  }}
                </label>
                <input
                  v-model="financialDocument.customText"
                  class="form-control fr-input validate-required"
                  :class="{
                    'fr-input--valid': valid,
                    'fr-input--error': errors[0]
                  }"
                  id="customText"
                  name="customText"
                  placeholder=""
                  type="text"
                  required
                />
                <span class="fr-error-text" v-if="errors[0]">{{
                  $t(errors[0])
                }}</span>
              </div>
            </validation-provider>
          </div>
        </div>
      </NakedCard>
    </ValidationObserver>
    <ProfileFooter @on-back="goBack" @on-next="goNext"></ProfileFooter>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import { DocumentType } from "df-shared/src/models/Document";
import DocumentInsert from "../share/DocumentInsert.vue";
import FileUpload from "../../uploads/FileUpload.vue";
import { UploadStatus } from "df-shared/src/models/UploadStatus";
import ListItem from "../../uploads/ListItem.vue";
import { DfFile } from "df-shared/src/models/DfFile";
import { DfDocument } from "df-shared/src/models/DfDocument";
import { Guarantor } from "df-shared/src/models/Guarantor";
import { extend } from "vee-validate";
import { RegisterService } from "../../../services/RegisterService";
import DfButton from "df-shared/src/Button/Button.vue";
import { ValidationObserver, ValidationProvider } from "vee-validate";
import { required, regex } from "vee-validate/dist/rules";
import WarningMessage from "df-shared/src/components/WarningMessage.vue";
import { DocumentTypeConstants } from "../share/DocumentTypeConstants";
import ConfirmModal from "df-shared/src/components/ConfirmModal.vue";
import { FinancialDocument } from "df-shared/src/models/FinancialDocument";
import { mapGetters, mapState } from "vuex";
import Modal from "df-shared/src/components/Modal.vue";
import GuarantorChoiceHelp from "../../helps/GuarantorChoiceHelp.vue";
import VGouvFrModal from "df-shared/src/GouvFr/v-gouv-fr-modal/VGouvFrModal.vue";
import ProfileFooter from "../../footer/ProfileFooter.vue";
import NakedCard from "df-shared/src/components/NakedCard.vue";
import BigRadio from "df-shared/src/Button/BigRadio.vue";
import cloneDeep from "lodash/cloneDeep";
import { AnalyticsService } from "../../../services/AnalyticsService";
import AllDeclinedMessages from "../share/AllDeclinedMessages.vue";
import { DocumentDeniedReasons } from "df-shared/src/models/DocumentDeniedReasons";
import TroubleshootingModal from "@/components/helps/TroubleshootingModal.vue";

extend("regex", {
  ...regex,
  message: "number-not-valid"
});

extend("required", {
  ...required,
  message: "field-required"
});

@Component({
  components: {
    AllDeclinedMessages,
    ValidationProvider,
    ValidationObserver,
    DocumentInsert,
    FileUpload,
    ListItem,
    DfButton,
    WarningMessage,
    ConfirmModal,
    Modal,
    GuarantorChoiceHelp,
    VGouvFrModal,
    ProfileFooter,
    NakedCard,
    BigRadio,
    TroubleshootingModal
  },
  computed: {
    ...mapState({
      selectedGuarantor: "selectedGuarantor"
    }),
    ...mapGetters({
      guarantorFinancialDocumentSelected: "guarantorFinancialDocumentSelected",
      guarantorFinancialDocuments: "guarantorFinancialDocuments"
    })
  }
})
export default class GuarantorFinancialDocumentForm extends Vue {
  @Prop() tenantId?: string;

  selectedGuarantor!: Guarantor;
  guarantorFinancialDocumentSelected!: FinancialDocument;
  financialDocument = new FinancialDocument();

  documentDeniedReasons = new DocumentDeniedReasons();
  documents = DocumentTypeConstants.GUARANTOR_FINANCIAL_DOCS;
  isDocDeleteVisible = false;
  selectedDoc?: FinancialDocument;
  isNoIncomeAndFiles = false;

  beforeMount() {
    this.financialDocument = {
      ...cloneDeep(this.guarantorFinancialDocumentSelected)
    };
    if (this.guarantorFinancialDocument()?.documentDeniedReasons) {
      this.documentDeniedReasons = cloneDeep(
        this.guarantorFinancialDocument()?.documentDeniedReasons
      );
    }
  }

  get documentStatus() {
    return this.guarantorFinancialDocument()?.documentStatus;
  }

  guarantorFinancialDocument() {
    return this.$store.getters.getGuarantorDocuments?.find((d: DfDocument) => {
      return d.id === this.financialDocument.id;
    });
  }

  onSelectChange() {
    if (this.financialDocument.id === null) {
      return false;
    }

    const doc = this.selectedGuarantor.documents?.find((d: DfDocument) => {
      return d.id === this.financialDocument.id;
    });
    if (doc === undefined) {
      return false;
    }

    this.isDocDeleteVisible =
      (doc.files?.length || 0) > 0 &&
      doc.documentSubCategory !== this.financialDocument.documentType.value;

    if (this.isDocDeleteVisible) {
      this.selectedDoc = this.financialDocument;
    }
    return false;
  }

  undoSelect() {
    if (this.selectedGuarantor.documents === null) {
      return;
    }
    const doc = this.selectedGuarantor.documents?.find((d: DfDocument) => {
      return d.id === this.selectedDoc?.id;
    });
    if (doc !== undefined) {
      const localDoc = this.documents.find((d: DocumentType) => {
        return d.value === doc.documentSubCategory;
      });
      if (localDoc !== undefined && this.selectedDoc) {
        this.selectedDoc.documentType = localDoc;
      }
    }
    this.isDocDeleteVisible = false;
  }

  async validSelect() {
    this.isDocDeleteVisible = false;
    if (this.selectedGuarantor.documents === null) {
      return;
    }
    const doc = this.selectedGuarantor.documents?.find((d: DfDocument) => {
      return d.id === this.selectedDoc?.id;
    });
    if (doc?.files !== undefined) {
      for (const f of doc.files) {
        if (f.id && this.selectedDoc) {
          await this.remove(this.selectedDoc, f, true);
        }
      }
    }
  }

  addFiles(f: FinancialDocument, fileList: File[]) {
    const nf = Array.from(fileList).map(f => {
      return { name: f.name, file: f, size: f.size };
    });
    f.files = [...f.files, ...nf];
    this.save();
  }

  resetFiles(f: FinancialDocument) {
    f.fileUploadStatus = UploadStatus.STATUS_INITIAL;
  }

  async save(): Promise<boolean> {
    const fieldName = "documents";
    const formData = new FormData();
    if (this.financialDocument.documentType.key === undefined) {
      return Promise.resolve(true);
    }

    if (this.financialDocument.id) {
      const original = this.$store.getters.guarantorFinancialDocuments?.find(
        (d: DfDocument) => {
          return d.id === this.financialDocument.id;
        }
      );
      if (
        this.financialDocument.noDocument === original?.noDocument &&
        this.financialDocument.monthlySum === original?.monthlySum &&
        this.financialDocument.files.length === original?.files.length
      ) {
        return Promise.resolve(true);
      }
    }
    AnalyticsService.registerFile("guarantor-financial");
    if (!this.financialDocument.noDocument) {
      if (!this.financialFiles().length) {
        Vue.toasted.global.max_file({
          message: this.$i18n.t("guarantorfinancialdocumentform.missing-file")
        });
        return Promise.reject(new Error("err"));
      }

      if (
        this.financialDocument.documentType.maxFileCount &&
        this.financialFiles().length >
          this.financialDocument.documentType.maxFileCount
      ) {
        Vue.toasted.global.max_file({
          message: this.$i18n.t("guarantorfinancialdocumentform.max-file", [
            this.financialFiles().length,
            this.financialDocument.documentType.maxFileCount
          ])
        });
        return Promise.reject(new Error("max-file"));
      }

      const newFiles = this.financialDocument.files.filter(f => {
        return !f.id;
      });
      Array.from(Array(newFiles.length).keys()).map(x => {
        const f: File = newFiles[x].file || new File([], "");
        formData.append(`${fieldName}[${x}]`, f, newFiles[x].name);
      });
    } else {
      if (this.financialFiles().length > 0) {
        this.isNoIncomeAndFiles = true;
        return Promise.reject(new Error("err"));
      }
    }

    const typeDocumentFinancial =
      this.financialDocument.documentType?.value || "";
    formData.append("typeDocumentFinancial", typeDocumentFinancial);

    formData.append(
      "noDocument",
      this.financialDocument.noDocument ? "true" : "false"
    );
    formData.append("customText", this.financialDocument.customText);

    if (this.financialDocument.monthlySum) {
      formData.append(
        "monthlySum",
        this.financialDocument.monthlySum.toString()
      );
    } else {
      return Promise.reject(new Error("err"));
    }
    if (this.financialDocument.id) {
      formData.append("documentId", this.financialDocument.id.toString());
    }
    if (this.financialDocument.customText != "") {
      formData.append("customText", this.financialDocument.customText);
    }

    this.financialDocument.fileUploadStatus = UploadStatus.STATUS_SAVING;
    formData.append("guarantorId", this.$store.getters.guarantor.id);
    if (this.tenantId) {
      formData.append("tenantId", this.tenantId);
    }
    const loader = this.$loading.show();
    const res = await this.$store
      .dispatch("saveGuarantorFinancial", formData)
      .then(() => {
        this.financialDocument = Vue.set(this, "financialDocument", {
          ...cloneDeep(this.guarantorFinancialDocumentSelected)
        });
        Vue.toasted.global.save_success();
        return Promise.resolve(true);
      })
      .catch(err => {
        this.financialDocument.fileUploadStatus = UploadStatus.STATUS_FAILED;
        if (err.response.data.message.includes("NumberOfPages")) {
          Vue.toasted.global.save_failed_num_pages();
        } else {
          Vue.toasted.global.save_failed();
        }
        return Promise.reject(new Error("err"));
      })
      .finally(() => {
        loader.hide();
      });
    return res;
  }

  financialFiles() {
    const newFiles = this.financialDocument.files.map((file: DfFile) => {
      return {
        documentSubCategory: this.financialDocument.documentType?.value,
        id: file.name,
        name: file.name,
        size: file.size
      };
    });
    const existingFiles =
      this.$store.getters.getGuarantorDocuments?.find((d: DfDocument) => {
        return d.id === this.financialDocument.id;
      })?.files || [];
    return [...newFiles, ...existingFiles];
  }

  async remove(f: FinancialDocument, file: DfFile, silent = false) {
    if (file.id) {
      await RegisterService.deleteFile(file.id, silent);
    } else {
      const firstIndex = f.files.findIndex(f => {
        return f.name === file.name && f.file === file.file && !f.id;
      });
      f.files.splice(firstIndex, 1);
    }
  }

  goBack() {
    this.$store.commit("selectGuarantorDocumentFinancial", undefined);
  }

  goNext() {
    this.save().then(() => {
      this.$store.commit("selectGuarantorDocumentFinancial", undefined);
    });
  }

  getCheckboxLabel(key: string) {
    if (key === "guarantor_salary") {
      return "noDocument-salary";
    }
    if (key === "pension") {
      return "noDocument-pension";
    }
    if (key === "rent") {
      return "noDocument-rent";
    }
    if (key === "scholarship") {
      return "noDocument-scholarship";
    }
    if (key === "social-service") {
      return "noDocument-social";
    }
    return "";
  }

  getCustomTextLabel(key: string) {
    if (key === "guarantor_salary") {
      return "customText-salary";
    }
    if (key === "pension") {
      return "customText-pension";
    }
    if (key === "rent") {
      return "customText-rent";
    }
    if (key === "scholarship") {
      return "customText-scholarship";
    }
    if (key === "social-service") {
      return "customText-social";
    }
    return "";
  }

  guarantorKey() {
    if (this.tenantId != null) {
      return "cotenant-guarantor";
    }
    return "guarantor";
  }
}
</script>

<style scoped lang="scss"></style>

