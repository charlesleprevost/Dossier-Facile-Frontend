<template>
  <div class="fr-container">
    <div class="fr-grid-row justify-content-center">
      <div class="fr-col-12">
        <div v-if="file.path">
          <img slot="image" v-auth-image="file.path" v-if="isImage()" :alt="$t('showdoc.preview')" />
          <PdfViewer
            :src="pdfContent"
            v-if="!isImage() && isLoaded"
          ></PdfViewer>
        </div>
        <div v-else>
          <div v-if="file.numberOfPages && file.numberOfPages > 0">
            {{ $t("showdoc.number-of-pages", [file.numberOfPages]) }}
          </div>
          <img slot="image" v-auth-image="file.preview" :alt="$t('showdoc.preview')" />
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { DfFile } from "df-shared/src/models/DfFile";
import { Component, Prop, Vue } from "vue-property-decorator";
import PdfViewer from "../../PdfViewer.vue";
import { ImageService } from "../../../services/ImageService";
import axios from "axios";

@Component({
  components: {
    PdfViewer
  }
})
export default class ShowDoc extends Vue {
  @Prop({ default: "" }) file!: DfFile;
  isLoaded = false;
  pdfContent = {};

  isImage() {
    return ImageService.isImage(this.file);
  }

  mounted() {
    if (this.file.path) {
      axios.get(this.file.path, { responseType: "blob" }).then(response => {
        const blob = new Blob([response.data], { type: "application/pdf" });
        this.pdfContent = window.URL.createObjectURL(blob);
        this.isLoaded = true;
      });
    }
  }
}
</script>

