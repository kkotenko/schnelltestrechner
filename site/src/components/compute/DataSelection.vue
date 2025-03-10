<template>
  <v-expansion-panel>
    <v-expansion-panel-header
      class="font-weight-bold text-uppercase"
      color="blue lighten-3"
    >
      Datenquelle
    </v-expansion-panel-header>
    <v-expansion-panel-content color="blue lighten-5" class="pt-4">
      <p>
        Die grundlegenden Qualitätswerte eines Tests - Sensititvität und Spezifität -
        lassen sich zwar im Labor bestimmen. Aber verschiedene Labore kommen zu
        unterschiedlichen Ergebnissen, teilweise abhängig von der Auswahl der Probanden
        oder der Art der Probenentnahme. Für die Berechnung ist es jedoch nötig, zunächst
        ein bestimmtes Ergebnis zu betrachten, das hier ausgewählt wird.
      </p>

      <!--
      <v-btn-toggle v-model="testsKind">
        <v-btn value="list">
          <span class="hidden-xs-only mr-4">Aus Liste wählen</span>
          <span class="hidden-sm-and-up mr-4">Liste</span>
          <v-icon> mdi-format-list-checkbox </v-icon>
        </v-btn>

        <v-btn value="input">
          <span class="hidden-xs-only mr-4">Eigene Werte eingeben</span>
          <span class="hidden-sm-and-up mr-4">Eingabe</span>
          <v-icon> mdi-pencil </v-icon>
        </v-btn>
      </v-btn-toggle>
      -->

      <p class="mt-4">
        Quelle der Angaben zu Sensitivität und Spezifität:
        <v-card>
          <v-data-table
            class="mt-4"
            dense
            v-model="selectedStudies"
            :items="studies"
            :headers="studyHeaders"
            show-select
            single-select
            locale="de-DE"
            mobile-breakpoint="360"
            :options="{ itemsPerPage: 10 }"
          >
            <template v-slot:item.sensitivity="{ item }">
              {{
                item.sensitivity
                  ? $options.filters.formatPercent(item.sensitivity.avg)
                  : ""
              }}
            </template>
            <template v-slot:item.specificity="{ item }">
              {{
                item.specificity
                  ? $options.filters.formatPercent(item.specificity.avg)
                  : ""
              }}
            </template>
            <template v-slot:item.author="{ item }">
              <a v-if="item.url" :href="item.url" target="_blank">{{
                item.author == "manufacturer"
                  ? "Herstellerangaben " + item.comment
                  : item.author
              }}</a>
              <span v-else>
                {{
                  item.author == "manufacturer"
                    ? "Herstellerangaben " + item.comment
                    : item.author
                }}
              </span>
            </template>
            <template v-slot:item.quality="{ item }">
              <v-tooltip left v-if="!item.author.startsWith('[')">
                <template v-slot:activator="{ on, attrs }">
                  <v-icon v-bind="attrs" v-on="on" small :color="getStudyColor(item)">
                    mdi-circle
                  </v-icon>
                </template>
                <span>{{ getStudyQualityText(item) }}</span>
              </v-tooltip>
              {{ getStudySampleSizeString(item) }}
            </template>
            <template v-slot:item.sample="{ item }">
              <v-tooltip left>
                <template v-slot:activator="{ on, attrs }">
                  <div v-bind="attrs" v-on="on">
                    {{ getSampleIcons(item) }}
                  </div>
                </template>
                <span>{{ getSampleText(item) }}</span>
              </v-tooltip>
            </template>
          </v-data-table>
        </v-card>
      </p>

      <!--
        <p>
          Werte innerhalb des 95%-Konfidenzintervalls:

          <v-btn-toggle v-model="confidence" class="mt-2">
            <v-btn value="min">
              <span class="hidden-xs-only">Niedrigste</span>
              <v-icon> mdi-format-vertical-align-bottom </v-icon>
            </v-btn>
            <v-btn value="avg">
              <span class="hidden-xs-only">Mittlere</span>
              <v-icon> mdi-format-vertical-align-center </v-icon>
            </v-btn>
            <v-btn value="max">
              <span class="hidden-xs-only">Höchste</span>
              <v-icon> mdi-format-vertical-align-top </v-icon>
            </v-btn>
          </v-btn-toggle>
        </p>
      -->

      <template v-if="studyId == 'ownValues'">
        <v-container fluid class="pt-8">
          <v-row>
            <v-col cols="12" sm="6" md="3" no-gutters>
              Sensitivität:
              <v-text-field v-model="sensitivityString" solo />
            </v-col>
            <v-col cols="12" sm="6" md="3" no-gutters>
              Spezifität:
              <v-text-field v-model="specificityString" solo />
            </v-col>
          </v-row>
          <v-row>
            <template v-if="detailedMode">
              <v-col cols="12" sm="6" md="3" no-gutters
                >Falsch-Negativ-Rate: {{ fnr | formatPercent }}
              </v-col>
              <v-col cols="12" sm="6" md="3" no-gutters
                >Falsch-Positiv-Rate: {{ fpr | formatPercent }}
              </v-col>
              <v-col cols="12" sm="6" md="3" no-gutters
                >Bayes-Faktor bei positivem Test:
                {{ bayesFactorPos | formatNumber }}</v-col
              >
              <v-col cols="12" sm="6" md="3" no-gutters
                >Bayes-Faktor bei negativem Test:
                {{ (1 / bayesFactorNeg) | formatNumber }}</v-col
              >
            </template>
          </v-row>
        </v-container>
      </template>

      <v-container fluid class="pt-8" v-else>
        <v-row>
          <v-col cols="12" sm="6" md="3" no-gutters>
            Sensitivität: {{ sensitivity | formatPercent }}
          </v-col>
          <v-col cols="12" sm="6" md="3" no-gutters>
            Spezifität: {{ specificity | formatPercent }}
          </v-col>
        </v-row>
        <v-row>
          <template v-if="detailedMode">
            <v-col cols="12" sm="6" md="3" no-gutters
              >Falsch-Negativ-Rate: {{ fnr | formatPercent }}
            </v-col>
            <v-col cols="12" sm="6" md="3" no-gutters
              >Falsch-Positiv-Rate: {{ fpr | formatPercent }}
            </v-col>
            <v-col cols="12" sm="6" md="3" no-gutters
              >Bayes-Faktor bei positivem Test: {{ bayesFactorPos | formatNumber }}</v-col
            >
            <v-col cols="12" sm="6" md="3" no-gutters
              >Bayes-Faktor bei negativem Test:
              {{ (1 / bayesFactorNeg) | formatNumber }}</v-col
            >
          </template>
        </v-row>
      </v-container>

      <v-alert
        icon="mdi-alert"
        dense
        outlined
        type="info"
        text
        class="mt-4"
        v-if="this.selectedTest != null && this.selectedTest.logisticRegression"
      >
        Dieser Test wurde in der Studie "Comparative sensitivity evaluation for 122
        CE-marked SARS-CoV-2 antigen rapid tests" von Scheiblauer et. al. (<a
          target="_blank"
          href="https://doi.org/10.1101/2021.05.11.21257016"
          >Link zum Preprint</a
        >) betrachtet. Dadurch liegen besonders detaillierte Daten zur Sensitivität vor,
        die in Abhängigkeit von der Sensititvität <i>betrachtet</i> werden können. Allerdings
        können diese Daten derzeit nicht direkt in die Berechnung übernommen werden.<br/>Die Berechnung der folgenden Werte ist noch in Entwicklung und als Vorläufig zu betrachten:
      </v-alert>

      <v-card v-if="this.selectedTest != null && this.selectedTest.logisticRegression">
        <v-card-title>Sensititvität nach Viruslast</v-card-title>
        <v-container fluid class="pt-8">
          <v-row>
            <v-col cols="12" sm="8" md="9" no-gutters>
              <v-slider
                v-model="ct"
                min="17"
                max="37"
                label="CT-Wert"
                :thumb-size="24"
                thumb-label="always"
                class="ml-4 mr-4 mt-4"
                tick-size="6"
                ticks="always"
                :tick-labels="[
                  '17',
                  '',
                  '',
                  '',
                  '',
                  '',
                  'hoch ansteckend',
                  '',
                  '',
                  '',
                  '',
                  'gering ansteckend',
                  '',
                  '',
                  '',
                  '',
                  'kaum ansteckend',
                  '',
                  '',
                  '',
                  '37',
                ]"
              ></v-slider>
            </v-col>
            <v-col cols="12" sm="4" md="3" no-gutters class="text-h3">
              {{ $options.filters.formatPercent(logarithmicSensitivity) }}
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="12" sm="8" md="9" no-gutters>
              Entspricht {{ $options.filters.formatLargeNumber(viralLoad) }} Genomkopien /
              ml
            </v-col>
            <v-col cols="12" sm="4" md="3" no-gutters> </v-col>
          </v-row>
        </v-container>
      </v-card>

      <!--
      <v-alert
        icon="mdi-alert"
        dense
        outlined
        type="warning"
        text
        class="mt-4"
        v-if="this.selectedTest != null && Number.isNaN(this.sensitivity)"
      >
        Zu diesem Test liegen uns keine Daten zu Sensitivität und Spezifität vom
        Hersterller vor. Bitte einen anderen Test auswählen oder eigene Werte eingeben.
      </v-alert>

      <v-alert
        icon="mdi-alert"
        dense
        outlined
        type="warning"
        text
        class="mt-4"
        v-if="peiAssumption"
      >
        Zu diesem Test liegen uns keine Daten zu Sensitivität und Spezifität vom
        Hersterller vor.
        <b>Für die Berechnung werden daher die Mindestkriterien des PEI angenommen.</b>
        Falls du die Daten kennst, z.B. aus dem Beipackzettel, kannst du diese selbst
        eingeben. In dem Fall freuen wir uns auch über eine Nachricht, damit wir die Daten
        bei uns einpflegen können.
      </v-alert>
      -->
    </v-expansion-panel-content>
  </v-expansion-panel>
</template>
<script>
//import Info from "../Info.vue";
import {
  getSampleIcons,
  getSampleText,
  getStudyColor,
  getStudySampleSizeString,
  getStudyQualityText,
} from "../../helpers.js";

export default {
  components: {
    //  Info,
  },
  name: "DataSelection",
  props: {
    selectedTest: Object,
    detailedMode: Boolean,
  },
  data: () => ({
    testsKind: "list",
    selectedStudies: [{ id: "nothing", author: "[nichts ausgewählt]" }],
    sensitivityString: "80,0 %",
    specificityString: "80,0 %",
    confidence: "avg",
    ct: 30,
    studyHeaders: [
      { text: "Autor_innen", value: "author", sortable: true },
      { text: "Studienqualität", value: "quality", sortable: true },
      { text: "Probenart", value: "sample", sortable: true },
      { text: "Sensitivität", value: "sensitivity", sortable: true },
      { text: "Spezifität", value: "specificity", sortable: true },
    ],
  }),
  computed: {
    selectedStudy() {
      return this.selectedStudies[0];
    },
    studyId() {
      if (this.selectedStudies.length == 0) {
        return null;
      }
      return this.selectedStudies[0].id;
    },
    studies() {
      const nothing = { id: "nothing", author: "[nichts ausgewählt]" };
      const ownValues = { id: "ownValues", author: "[eigene Werte eingeben]" };
      const minPei = {
        id: "minPei",
        author: "[Paul-Ehrlich-Institut - allg. Mindestwerte]",
        sensitivity: { avg: 0.8 },
        specificity: { avg: 0.97 },
        url:
          "https://www.pei.de/SharedDocs/Downloads/DE/newsroom/dossiers/mindestkriterien-sars-cov-2-antigentests-01-12-2020.pdf?__blob=publicationFile&v=6",
      };
      if (this.selectedTest) {
        let pei = [];
        if (this.selectedTest.pei) {
          pei = [minPei];
        }
        let studies = Object.values(this.selectedTest.studies);
        studies.sort(
          (a, b) =>
            (b.author == "manufacturer" ? 1 : 0) - (a.author == "manufacturer" ? 1 : 0)
        );
        return [nothing, ownValues, ...pei, ...studies];
      }
      return [nothing, ownValues, minPei];
    },
    peiAssumption() {
      return (
        this.selectedTest &&
        this.studyId != "ownValues" &&
        !this.selectedTest.studies[this.studyId] &&
        this.selectedTest.pei
      );
    },
    logarithmicSensitivity() {
      return (
        1 /
        (1 +
          Math.exp(
            -(
              this.selectedTest.logisticRegression.intercept +
              this.selectedTest.logisticRegression.coef * this.ct
            )
          ))
      );
    },
    viralLoad() {
      return 220000000000000 / Math.pow(2, this.ct);
    },
    sensitivity() {
      if (this.studyId == "minPei") {
        return 0.8;
      } else if (this.studyId == "ownValues") {
        return (
          parseFloat(this.sensitivityString.replace(",", ".").replace("%", "")) / 100.0
        );
      } else {
        if (!this.selectedTest) {
          return Number.NaN;
        }
        if (this.selectedTest.studies[this.studyId]) {
          return this.selectedTest.studies[this.studyId].sensitivity[this.confidence];
        } else {
          return Number.NaN;
        }
      }
    },
    specificity() {
      if (this.studyId == "minPei") {
        return 0.97;
      } else if (this.studyId == "ownValues") {
        return (
          parseFloat(this.specificityString.replace(",", ".").replace("%", "")) / 100.0
        );
      } else {
        if (!this.selectedTest) {
          return Number.NaN;
        }
        if (this.selectedTest.studies[this.studyId]) {
          return this.selectedTest.studies[this.studyId].specificity[this.confidence];
        } else {
          return Number.NaN;
        }
      }
    },
  },
  methods: {
    getSampleIcons,
    getSampleText,
    getStudyColor,
    getStudySampleSizeString,
    getStudyQualityText,
  },
  watch: {
    sensitivity: function (newVal) {
      this.$emit("sensitivity", newVal);
    },
    specificity: function (newVal) {
      this.$emit("specificity", newVal);
    },
  },
};
</script>
