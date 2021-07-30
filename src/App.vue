<template>
  <div id="app">
    <div class="p-4 d-print-none container">
      <div class="border rounded p-3 shadow-sm mb-3">
        <img
          src="https://www.capiotec.fr/web/image/res.company/1/logo?unique=3fb330c"
          class="img-fluid"
          style="max-height: 60px"
        />
        <h1 class="mb-1">
          Bienvenue dans le générateur d'étiquettes de Capiotec
        </h1>
        <p class="text-muted mb-0">version 0.1 par Quentin Gabriele</p>
      </div>
      <div class="d-flex flex-wrap rounded shadow-sm border">
        <div class="p-3 w-md-25">
          <h5 class="fw-bold">🔎 Ajouter des articles</h5>
          <p class="mb-0">En chargeant un ficher:</p>
          <input
            @change="onInputFile"
            type="file"
            name="csv"
            accept="text/csv"
          />
          <hr />
          <p class="mb-0">Manuellement</p>
          <p class="mb-2">
            <button @click="onAdd" class="has-shadow">➕ Ajouter</button>
          </p>
          <p class="mb-0 text-muted text-small fs-6">
            tips: supprimez le fond de vos photos rapidement:
            <a
              href="https://www.photoroom.com/background-remover/"
              class="text-reset"
              target="_blank"
            >
              PhotoRoom.com
            </a>
          </p>
        </div>
        <div class="p-3 w-md-25">
          <h5 class="fw-bold">👀 Choisissez la taille</h5>
          <div class="field">
            <input
              v-model="type"
              id="type-large"
              type="radio"
              name="type"
              value="large"
            />
            <label for="type-large">Bac</label>
          </div>
          <div class="field">
            <input
              v-model="type"
              id="type-small"
              type="radio"
              name="type"
              value="small"
            />
            <label for="type-small">Visserie large</label>
          </div>
          <div class="field">
            <input
              v-model="type"
              id="type-tiny"
              type="radio"
              name="type"
              value="tiny"
            />
            <label for="type-tiny">Visserie petite</label>
          </div>
        </div>
        <div class="p-3 w-md-25">
          <h5 class="fw-bold">🧾 Le nombre d'étiquette par page</h5>
          <div class="field">
            <label for="itemsPerPage" class="w-100">
              Nombre d'elements par page
            </label>
            <input
              style="max-width: 100px"
              v-model="itemsPerPage"
              type="number"
              interval="1"
              name="itemsPerPage"
            />
          </div>
        </div>
        <div class="p-3 w-md-25">
          <h5 class="fw-bold">🖨 Puis imprimer la page</h5>
          <button @click="print()" class="has-shadow">🖨 Imprimer</button>

          <hr />
          <div class="field">
            <input
              v-model="preview"
              id="preview-edit"
              type="radio"
              name="preview"
              :value="false"
            />
            <label for="preview-edit">Modifiable</label>
          </div>
          <div class="field">
            <input
              v-model="preview"
              id="preview-printer"
              type="radio"
              name="preview"
              :value="true"
            />
            <label for="preview-printer">Aperçu avant impression</label>
          </div>
          <hr>
          <div>
            <button @click="onExport" class="has-shadow">💾 Exporter</button>
          </div>
        </div>
      </div>
    </div>
    <div
      class="page"
      :class="{ preview: preview }"
      v-for="(page, pageIndex) in paginatedItems"
      :key="pageIndex"
    >
      <div v-for="(item, index) in page" :key="'' + index + pageIndex">
        <div class="d-print-none">
          <button v-if="item.delete" class="button" @click="onDelete(item)">
            🕳 Supprimer
          </button>
          <button
            v-if="item.duplicate"
            class="button"
            @click="onDuplicate(item)"
          >
            🌟 Dupliquer
          </button>
          <button
            v-if="item.duplicate"
            class="button"
            @click="onChangeType(item)"
          >
            {{ item.type || type }}
          </button>
        </div>

        <div class="etiquette" :class="[item.type || type]">
          <div class="etiquette-img">
            <img v-if="item.img" class="img-cover" :src="item.img" />
            <input
              class="d-print-none"
              type="file"
              @input="onAddImage(item, $event)"
            />
          </div>
          <div class="etiquette-content">
            <div class="etiquette-title">
              <!-- editor -->
              <div class="d-print-none">
                <input
                  v-if="item.type == 'large' || (!item.type && type == 'large')"
                  v-model="item.ref"
                  class="transparent p-0"
                  :style="{ fontSize: computeFontSize(item) }"
                />
                <input
                  v-else
                  v-model="item.name"
                  class="transparent p-0"
                  :style="{ fontSize: computeFontSize(item) }"
                />
              </div>
              <!-- printable -->
              <div
                class="no-screen"
                :style="{ fontSize: computeFontSize(item) }"
              >
                <span
                  v-if="item.type == 'large' || (!item.type && type == 'large')"
                >
                  {{ item.ref }}
                </span>
                <span v-else>
                  {{ item.name }}
                </span>
              </div>
            </div>

            <div class="etiquette-subtitle">
              <textarea
                v-if="item.type == 'large' || (!item.type && type == 'large')"
                v-model="item.name"
                rows="3"
                class="d-print-none transparent p-0"
              ></textarea>
              <div v-else class="d-flex d-print-none w-100">
                <div class="flex-grow-1">
                  <input class="transparent p-0" v-model="item.ref" />
                </div>
                <div class="flex-grow-1">
                  <input
                    class="transparent p-0"
                    v-model="item.position"
                    style="max-width: 7mm"
                  />
                </div>
              </div>

              <div class="no-screen w-100">
                <span
                  v-if="item.type == 'large' || (!item.type && type == 'large')"
                >
                  {{ item.name }}
                </span>
                <div v-else class="d-flex w-100">
                  <span class="flex-grow-1">
                    {{ item.ref }}
                  </span>
                  <span class="" style="max-width: 7mm">
                    {{ item.position }}
                  </span>
                </div>
              </div>
            </div>
            <div class="etiquette-meta">
              <div
                class="border-bottom border-dark d-flex text-muted text-small"
              >
                <div class="cell" v-if="item.price">Prix</div>
                <div class="cell" v-else>Poids</div>
                <div class="cell">min</div>
                <div class="cell">max</div>
              </div>
              <div class="d-flex">
                <div class="cell" v-if="item.price">
                  <input
                    type="text"
                    v-model="item.price"
                    class="d-print-none transparent p-0"
                  />
                  <span class="no-screen">
                    {{ item.price }}
                  </span>
                </div>
                <div class="cell" v-else>
                  <input
                    type="text"
                    v-model="item.weight"
                    class="d-print-none transparent p-0"
                  />
                  <span class="no-screen">
                    {{ item.weight }}
                  </span>
                </div>
                <div class="cell">
                  <input
                    type="text"
                    v-model="item.min"
                    class="d-print-none transparent p-0"
                  />
                  <span class="no-screen">
                    {{ item.min }}
                  </span>
                </div>
                <div class="cell">
                  <input
                    type="text"
                    v-model="item.max"
                    class="d-print-none transparent p-0"
                  />
                  <span class="no-screen">
                    {{ item.max }}
                  </span>
                </div>
              </div>
            </div>
          </div>
        </div>
        <!-- etiquette -->
      </div>
    </div>
    <!-- page -->
  </div>
  <!-- app -->
</template>

<script>
import Papa from "papaparse";

export default {
  directives: {},
  data() {
    return {
      type: "large",
      itemsPerPage: 8,
      preview: false,
      placeholder: [
        {
          delete: false,
          duplicate: false,
          img:
            "https://www.capiotec.fr/web/image/product.template/1766/image?unique=eba56e7",
          ref: "C0032-STD",
          name: "PANNEAU DE CONSIGNATION MOBILE A4",
          min: 0,
          max: 400,
          weight: 0.005,
        },
      ],
      items: [],
    };
  },
  watch: {
    type(newVal) {
      this.items.forEach(function (item) {
        item.type = null;
      });
    },
  },
  computed: {
    itemsToShow() {
      return this.items.length > 0 ? this.items : this.placeholder;
    },
    numberOfPages() {
      return Math.ceil(this.itemsToShow.length / this.itemsPerPage);
    },
    paginatedItems() {
      var pages = [];

      for (var pageIndex = 1; pageIndex <= this.numberOfPages; pageIndex++) {
        var start = (pageIndex - 1) * this.itemsPerPage;
        var end = start + this.itemsPerPage;
        pages.push(this.itemsToShow.slice(start, end));
      }
      return pages;
    },
  },
  methods: {
    print() {
      window.print();
    },
    onAddImage(item, e) {
      var files = e.target.files || e.dataTransfer.files;
      item.img = URL.createObjectURL(files[0]);
    },
    computeFontSize(item) {
      var type = item.type || this.type;
      var text = type === "large" ? item.ref : item.name;
      if (type === "small") return this.computeSmallFontSize(text);
      if (type === "tiny") return this.computeTinyFontSize(text);
      var size = 7.5;
      let length = text.length;

      if (length <= 10) {
        size -= 0.4;
      } else if (length <= 11) {
        size -= 1;
      } else if (length <= 12) {
        size -= 1.2;
      } else if (length <= 13) {
        size -= 1.4;
      } else {
        size -= 1.8;
      }

      return size + "mm";
    },
    computeSmallFontSize(text) {
      var size = 6;
      let length = text.length;

      if (length <= 17) {
        size -= 0.4;
      } else if (length <= 18) {
        size -= 0.8;
      } else if (length <= 19) {
        size -= 1;
      } else if (length <= 20) {
        size -= 1.4;
      } else {
        size -= 1.8;
      }

      return size + "mm";
    },
    computeTinyFontSize(text) {
      var size = 6;
      let length = text.length;
      if (length <= 9) {
        size -= 0;
      } else if (length <= 10) {
        size -= 0.4;
      } else if (length <= 11) {
        size -= 0.8;
      } else if (length <= 12) {
        size -= 1;
      } else if (length <= 13) {
        size -= 1.4;
      } else {
        size -= 1.8;
      }

      return size + "mm";
    },
    onAdd() {
      this.items.push({
        delete: true,
        duplicate: true,
        img: null,
        ref: "Ref",
        name: "nom",
        min: "-",
        max: "-",
        weight: null,
        type: null,
        price: "",
        position: "",
      });
    },
    onChangeType(item) {
      if (item.type === "large") item.type = "small";
      else if (item.type === "small") item.type = "tiny";
      else if (item.type === "tiny") item.type = "large";
      else item.type = "small";
    },
    onDelete(item) {
      var index = this.items.findIndex((el) => el.ref === item.ref);
      this.items.splice(index, 1);
    },
    onDuplicate(item) {
      var index = this.items.findIndex((el) => el.ref === item.ref);
      this.items.splice(index, 0, { ...this.items[index] });
    },
    onInputFile(e) {
      var files = e.target.files || e.dataTransfer.files;

      Array.from(files).forEach(async (file)=>{
        if(file.type === "application/json"){
          this.onImportJson(
            JSON.parse(
              await file.text()
            )
          );
        }else{
          this.parseCSV(file);
        }
      });
    },
    findIndexOfColumn(labels, name) {
      return labels.indexOf(name);
    },
    indexOfColumns(labels) {
      let objectOfLabels = {};
      labels.forEach((item, index) => {
        objectOfLabels[item] = index;
      });
      return objectOfLabels;
    },
    onImportJson(data){
      this.items = this.items.concat(data);
    },
    onExport(){
      const data = JSON.stringify(this.items);
      const fileName = "export-"+ (new Date()).toLocaleString()+'.json';
      const a = document.createElement("a");
      const file = new Blob([data], { type: "application/json" });
      a.href = URL.createObjectURL(file);
      a.download = fileName;
      a.click();
    },
    parseCSV(file) {
      Papa.parse(file, {
        complete: (results) => {
          //remove first row = remove columns labels
          let labels = results.data.shift();
          let indexes = this.indexOfColumns(labels);

          //remove item without Internal reference
          results.data = results.data.filter((item) => !!item[1]);

          //add items to the list
          this.items = this.items.concat(
            results.data.map((item) => {
              let img = item[indexes["Image de grande taille"]];
              let ref = item[indexes["Référence interne"]];
              let name = item[indexes["Nom"]];
              let min =
                item[indexes["Règles de stock minimum/Quantité minimum"]];
              let max =
                item[indexes["Règles de stock minimum/Quantité maximale"]];
              let weight = item[indexes["Poids"]];
              let price = item[indexes["Prix de vente"]];

              return {
                delete: true,
                duplicate: true,
                type: null,
                img: img ? "data:image/png;base64," + img : null,
                ref: ref,
                name: name,
                min: min ? parseInt(min, 10) : "-",
                max: max ? parseInt(max, 10) : "-",
                weight: weight && ref.includes("CCF") ? weight : null,
                price:
                  price && ref.includes("CCF")
                    ? null
                    : Math.round(price * 100) / 100 + "€",
              };
            })
          );
        },
      });
    },
  },
  created() {},
};
</script>

<!-- Use preprocessors via the lang attribute! e.g. <style lang="scss"> -->
<style lang="scss">
body {
  font-family: Arial, system-ui, -apple-system, "Segoe UI", Roboto,
    "Helvetica Neue", Arial, "Noto Sans", "Liberation Sans", sans-serif,
    "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
}
.container {
  margin: auto;
  max-width: 1400px;
}
@page {
  margin: 4mm;
}

@media screen {
  .page:not(.preview) {
    .no-screen {
      display: none;
    }
  }
  .page.preview {
    .d-print-none {
      display: none !important;
    }
  }
}
@media (min-width:960px){
  .w-md-25{
    min-width:25%;
    width:25%;
    max-width:25%;
  }
}
button {
  display: inline-block;
  font-weight: 400;
  line-height: 1.5;
  color: #212529;
  text-align: center;
  text-decoration: none;
  vertical-align: middle;
  cursor: pointer;
  -webkit-user-select: none;
  -moz-user-select: none;
  user-select: none;
  padding: 0.255rem 0.75rem;
  font-size: 1rem;
  border-radius: 3px;
  border: solid 2px #212529;

  transition: all ease 0.1s;
  &.has-shadow {
    box-shadow: 2px 3px 0px 0px black;
  }
  &:active {
    box-shadow: 0px 0px 0px 0px black;
  }
}
input,
textarea {
  max-width: 100%;
  width: 100%;
}
input[type="radio"] {
  width: auto;
}
input.transparent,
textarea.transparent {
  border: none;
  padding: 0;
}
@media screen {
  .page {
    margin: auto;
    max-width: 210mm;
    outline: dashed 1px black;
    padding: 4mm;
  }
}

.page {
  max-height: 297mm;
  height: calc(297mm - 4mm * 2);
  display: flex;
  flex-wrap: wrap;
  align-content: flex-start;
}

img {
  height: auto;
  max-width: 100%;
}

.text-small {
  font-size: 4mm;
}

.etiquette {
  border: solid 1px black;
  width: 10cm;
  max-width: 10cm;
  min-width: 10cm;
  height: 6.5cm;
  display: flex;
  overflow: hidden;
  .etiquette-title {
    input {
      font-weight: bold;
    }
    font-weight: bold;
  }
  .etiquette-subtitle {
    flex-grow: 1;
    display: flex;
    align-items: center;
  }
  .etiquette-img img {
    height: 100%;
    width: 100%;
    object-fit: contain;
  }
  .etiquette-img {
    overflow: auto;
  }
  .etiquette-img,
  .etiquette-content {
    height: 100%;
    width: 50%;
    min-width: 50%;
    max-width: 50%;
  }
  .etiquette-content {
    flex-direction: column;
  }
}
.etiquette.large {
  .etiquette-content {
    display: flex;
    border-left: solid 1px black;
  }
  .etiquette-title {
    border-bottom: solid 1px black;
    padding: 1mm;
    font-size: 7.5mm;
  }
  .etiquette-subtitle {
    padding: 1mm;
    font-size: 4.5mm;
    border-bottom: solid 1px black;
  }
}
.etiquette.small {
  padding-left: 2mm;
  padding-right: 2mm;
  padding-bottom: 2mm;
  width: 63mm;
  max-width: 63mm;
  min-width: 63mm;
  height: 16mm;
  .etiquette-content {
    width: 100%;
    min-width: 100%;
    max-width: 100%;
    flex-wrap: wrap;
  }
  .etiquette-subtitle {
    font-size: 4mm;
  }
  .etiquette-img {
    display: none;
  }
  .etiquette-meta {
    display: none;
  }
}
.etiquette.tiny {
  padding-left: 2mm;
  padding-right: 2mm;
  padding-bottom: 2mm;
  width: 43mm;
  max-width: 43mm;
  min-width: 43mm;
  height: 16mm;
  .etiquette-content {
    width: 100%;
    min-width: 100%;
    max-width: 100%;
  }
  .etiquette-subtitle {
    font-size: 4mm;
  }
  .etiquette-img {
    display: none;
  }
  .etiquette-meta {
    display: none;
  }
}
.cell {
  flex-grow: 1;
  flex-shrink: 1;
  flex-basis: 0;
  padding: 1mm;
}
.cell:not(:last-child):not(:first-child) {
  border-left: solid 1px black;
  border-right: solid 1px black;
}
</style>
