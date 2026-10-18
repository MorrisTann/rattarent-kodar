<script setup>
import { computed, nextTick, ref, watch } from "vue";

const bikes = [
  {
    id: "3",
    name: "Kodar 3",
    price: 17.9,
    kind: "Lihtsad linnasõidud",
    description: "Lihtne ja töökindel linnaratas.",
    gears: "3 käiku",
    feature: "Linnasõiduks",
    kit: "Hooldus ja lukk",
    image: `${import.meta.env.BASE_URL}bike-3.svg`,
  },
  {
    id: "7",
    name: "Kodar 7",
    price: 24.9,
    kind: "Igapäevane linnasõit",
    description: "Mugav kaaslane kooli, tööle ja linna.",
    gears: "7 käiku",
    feature: "Linnasõiduks",
    kit: "Tuled ja lukk",
    image: `${import.meta.env.BASE_URL}bike-7.svg`,
    popular: true,
  },
  {
    id: "E",
    name: "Kodar E",
    price: 39.9,
    kind: "Pikemad linnasõidud",
    description: "Elektriline abi pikemateks sõitudeks.",
    gears: "Elektriline abi",
    feature: "Pikemateks sõitudeks",
    kit: "Tuled ja lukk",
    image: `${import.meta.env.BASE_URL}bike-e.svg`,
  },
];

const currentPage = ref("home");
const selectedId = ref("7");
const selectedBike = computed(
  () => bikes.find((bike) => bike.id === selectedId.value) || bikes[1],
);
const height = ref(168);
const selectedSize = ref("M");
const sizes = [
  { code: "S", min: 155, max: 165, label: "155–165 cm" },
  { code: "M", min: 165, max: 175, label: "165–175 cm" },
  { code: "L", min: 175, max: 190, label: "175–190 cm" },
];
const recommendedSize = computed(() => {
  const h = Number(height.value);
  if (!Number.isFinite(h) || h < 155 || h > 190) return null;
  if (h < 165) return "S";
  if (h <= 175) return "M";
  return "L";
});
watch(recommendedSize, (newSize) => {
  if (newSize) selectedSize.value = newSize;
});

const fullName = ref("");
const email = ref("");
const phone = ref("");
const pickupDate = ref(makeDate(2));
const pickupTime = ref("16:30");
const minDate = makeDate(0);
const confirmationNumber = ref("");
const formError = ref("");

function makeDate(offset) {
  const date = new Date();
  date.setDate(date.getDate() + offset);
  return [
    date.getFullYear(),
    String(date.getMonth() + 1).padStart(2, "0"),
    String(date.getDate()).padStart(2, "0"),
  ].join("-");
}
function euro(price) {
  return price.toFixed(2).replace(".", ",") + " €";
}
function dateText(date) {
  if (!date) return "";
  const [y, m, d] = date.split("-").map(Number);
  return new Intl.DateTimeFormat("et-EE", {
    day: "numeric",
    month: "long",
    year: "numeric",
  }).format(new Date(y, m - 1, d));
}
function navigate(page) {
  currentPage.value = page;
  nextTick(() => window.scrollTo({ top: 0, behavior: "instant" }));
}
function chooseBike(bike, destination = "detail") {
  selectedId.value = bike.id;
  navigate(destination);
}
async function showHowItWorks() {
  navigate("home");
  await nextTick();
  document
    .querySelector(".how-section")
    ?.scrollIntoView({ behavior: "smooth", block: "start" });
}
function goBack() {
  const prior = {
    bikes: "home",
    detail: "bikes",
    size: "detail",
    checkout: "size",
    success: "home",
  };
  navigate(prior[currentPage.value] || "home");
}
function reset() {
  selectedId.value = "7";
  selectedSize.value = "M";
  currentPage.value = "home";
  confirmationNumber.value = "";
  formError.value = "";
  nextTick(() => window.scrollTo(0, 0));
}
function confirmBooking() {
  formError.value = "";
  if (!fullName.value.trim() || !email.value.trim() || !phone.value.trim()) {
    formError.value = "Palun täida kõik kontaktandmed.";
    return;
  }
  if (!/^\S+@\S+\.\S+$/.test(email.value)) {
    formError.value = "Palun sisesta korrektne e-posti aadress.";
    return;
  }
  if (
    !pickupDate.value ||
    pickupDate.value < makeDate(0) ||
    !pickupTime.value
  ) {
    formError.value = "Palun vali sobiv tulevane kättesaamisaeg.";
    return;
  }
  confirmationNumber.value = `KOD-${Math.floor(1000 + Math.random() * 9000)}`;
  navigate("success");
}
</script>

<template>
  <div class="site-shell">
    <div class="site">
      <header
        class="site-header"
        :class="{
          'inner-header': currentPage !== 'home' && currentPage !== 'success',
        }"
      >
        <div class="header-left">
          <button
            v-if="currentPage !== 'home' && currentPage !== 'success'"
            class="back-button"
            type="button"
            @click="goBack"
            aria-label="Tagasi eelmisele lehele"
          >
            <svg viewBox="0 0 24 24" aria-hidden="true">
              <path d="m15 18-6-6 6-6" />
            </svg>
          </button>
          <button
            class="wordmark"
            type="button"
            @click="reset"
            aria-label="Kodar avalehele"
          >
            K<span>O</span>DAR
          </button>
        </div>
        <nav class="desktop-nav" aria-label="Peamenüü">
          <button
            type="button"
            :class="{ active: currentPage === 'home' }"
            @click="navigate('home')"
          >
            Avaleht
          </button>
          <button
            type="button"
            :class="{
              active: currentPage === 'bikes' || currentPage === 'detail',
            }"
            @click="navigate('bikes')"
          >
            Rattapaketid
          </button>
          <button type="button" @click="showHowItWorks">
            Kuidas see töötab?
          </button>
        </nav>
        <span class="site-city">TARTU</span>
      </header>

      <main :key="currentPage" class="page page-enter">
        <!-- 1. AVALEHT -->
        <template v-if="currentPage === 'home'">
          <section class="home-hero content-pad">
            <p class="location">
              <svg viewBox="0 0 24 24" aria-hidden="true">
                <path d="M12 21s7-6.1 7-12a7 7 0 1 0-14 0c0 5.9 7 12 7 12Z" />
                <circle cx="12" cy="9" r="2.5" />
              </svg>
              Tartu teenindussala
            </p>
            <h1>Sinu ratas.<br /><span>Meie mure.</span></h1>
            <p class="hero-description">
              Paindlik rattatellimus koos hooldusega. Sõida nii kaua, kui
              soovid.
            </p>
            <button
              class="primary-button"
              type="button"
              @click="navigate('bikes')"
            >
              Vali ratas <span aria-hidden="true">→</span>
            </button>
            <div class="home-cards" aria-label="Rattapakettide kiire ülevaade">
              <button
                v-for="bike in bikes"
                :key="bike.id"
                type="button"
                class="home-card"
                :class="{ featured: bike.popular }"
                @click="chooseBike(bike)"
              >
                <span v-if="bike.popular" class="mini-ribbon">POPULAARNE</span>
                <img :src="bike.image" :alt="bike.name + ' jalgratas'" />
                <strong>{{ bike.name }}</strong>
                <small>{{ euro(bike.price) }}<span> / kuus</span></small>
              </button>
            </div>
          </section>
          <section class="how-section content-pad">
            <p class="eyebrow">KUIDAS SEE TÖÖTAB</p>
            <h2>Kolm sammu, rohkem ei midagi</h2>
            <div class="steps">
              <div
                v-for="(step, index) in [
                  'Vali ratas',
                  'Saa kätte',
                  'Sõida muretult',
                ]"
                :key="step"
                class="step"
              >
                <div class="step-number">{{ index + 1 }}</div>
                <strong>{{ step }}</strong>
              </div>
            </div>
          </section>
          <div class="home-bottom"></div>
        </template>

        <!-- 2. PAKETID -->
        <section
          v-else-if="currentPage === 'bikes'"
          class="content-pad inner-page"
        >
          <p class="eyebrow">RATTAPAKETID</p>
          <h1 class="page-title">Leia oma<br />igapäevaratas</h1>
          <p class="page-subtitle">Kuutasu sees on alati korraline hooldus.</p>
          <div class="bike-list">
            <article
              v-for="bike in bikes"
              :key="bike.id"
              class="bike-list-card"
              :class="{ highlighted: bike.popular }"
            >
              <span v-if="bike.popular" class="ribbon">KÕIGE POPULAARSEM</span>
              <div class="bike-list-image">
                <img :src="bike.image" :alt="bike.name + ' jalgratas'" />
              </div>
              <div class="bike-list-info">
                <div class="bike-list-top">
                  <h2>{{ bike.name }}</h2>
                  <div class="price">
                    {{ euro(bike.price) }}<small>/ kuu</small>
                  </div>
                </div>
                <p>{{ bike.kind }}</p>
                <small
                  >{{ bike.gears }} <span>·</span>
                  {{
                    bike.kit === "Hooldus ja lukk"
                      ? bike.kit
                      : "Hooldus, " + bike.kit.toLowerCase()
                  }}</small
                >
                <button
                  type="button"
                  class="card-cta"
                  @click="chooseBike(bike)"
                >
                  Vaata lähemalt <span aria-hidden="true">→</span>
                </button>
              </div>
            </article>
          </div>
        </section>

        <!-- 3. RATTA DETAIL -->
        <section
          v-else-if="currentPage === 'detail'"
          class="content-pad inner-page detail-page"
        >
          <div class="bike-showcase">
            <span v-if="selectedBike.popular" class="ribbon"
              >KÕIGE POPULAARSEM</span
            >
            <img
              :src="selectedBike.image"
              :alt="selectedBike.name + ' jalgratas, külgvaade'"
            />
            <span class="bike-showcase-caption"
              >{{ selectedBike.name.toUpperCase() }} ·
              {{ selectedBike.kind.toUpperCase() }}</span
            >
          </div>
          <p class="eyebrow detail-eyebrow">SINULE, KES SÕIDAD IGA PÄEV</p>
          <div class="detail-heading">
            <h1>{{ selectedBike.name }}</h1>
            <div class="price detail-price">
              {{ euro(selectedBike.price) }}<small>/ kuu</small>
            </div>
          </div>
          <div class="benefits-grid">
            <div class="benefit">
              <span class="benefit-icon">{{
                selectedBike.id === "E"
                  ? "⚡"
                  : selectedBike.id === "3"
                    ? "3"
                    : "7"
              }}</span
              ><strong>{{ selectedBike.gears }}</strong
              ><small>{{
                selectedBike.id === "E" ? "Lihtsamad tõusud" : "Tartu tõusudeks"
              }}</small>
            </div>
            <div class="benefit">
              <span class="benefit-icon"
                ><svg viewBox="0 0 24 24">
                  <path d="m12 3 8 9-8 9-8-9Z" /></svg></span
              ><strong>{{ selectedBike.feature }}</strong
              ><small>Kerge ja mugav</small>
            </div>
            <div class="benefit">
              <span class="benefit-icon"
                ><svg viewBox="0 0 24 24">
                  <path d="m5 12 4 4L19 6" /></svg></span
              ><strong>Valmis sõiduks</strong
              ><small>{{ selectedBike.kit }}</small>
            </div>
          </div>
          <div class="details-list">
            <p>
              <span>✓</span> <strong>Hooldus on hinnas</strong> kogu tellimuse
              jooksul
            </p>
            <p><span>✓</span> Teenindus <strong>48 tunni jooksul</strong></p>
            <p>
              <span>✓</span> Lõpeta <strong>30-päevase etteteatamisega</strong>
            </p>
          </div>
          <button
            class="primary-button"
            type="button"
            @click="navigate('size')"
          >
            Vali {{ selectedBike.name }} <span aria-hidden="true">→</span>
          </button>
        </section>

        <!-- 4. RAAMISUURUS -->
        <section
          v-else-if="currentPage === 'size'"
          class="content-pad inner-page size-page"
        >
          <p class="eyebrow">RAAMISUURUS</p>
          <h1 class="page-title">Valime sulle sobiva</h1>
          <p class="page-subtitle">Sinu pikkuse järgi on valik lihtne.</p>
          <div class="size-layout">
            <div class="size-context">
              <div class="selected-summary">
                <div class="summary-picture">
                  <img :src="selectedBike.image" :alt="selectedBike.name" />
                </div>
                <div class="summary-name">
                  <small>Valitud ratas</small
                  ><strong>{{ selectedBike.name }}</strong>
                </div>
                <div class="summary-price">
                  {{ euro(selectedBike.price) }}<small>/ kuu</small>
                </div>
              </div>
              <label class="height-card" for="height-input">
                <span class="height-icon"
                  ><span></span><span></span><span></span
                ></span>
                <span
                  ><small>Sinu pikkus (cm)</small
                  ><input
                    id="height-input"
                    v-model.number="height"
                    aria-label="Sinu pikkus sentimeetrites"
                    type="number"
                    min="140"
                    max="210"
                    step="1"
                    inputmode="numeric"
                /></span>
              </label>
            </div>
            <div class="size-picker">
              <div class="size-head">
                <strong>Vali suurus</strong
                ><span v-if="recommendedSize"
                  >{{ recommendedSize }} on sulle soovitatud</span
                >
              </div>
              <div class="size-options">
                <button
                  v-for="size in sizes"
                  :key="size.code"
                  type="button"
                  class="size-option"
                  :class="{ selected: selectedSize === size.code }"
                  :aria-pressed="selectedSize === size.code"
                  @click="selectedSize = size.code"
                >
                  <span
                    v-if="recommendedSize === size.code"
                    class="recommend-tag"
                    >SOOVITUS</span
                  >
                  <strong>{{ size.code }}</strong
                  ><small>{{ size.label }}</small>
                </button>
              </div>
              <p class="size-footnote" v-if="recommendedSize">
                <span>✓</span> {{ selectedSize }} sobib pikkusele
                {{ sizes.find((s) => s.code === selectedSize)?.label }}
              </p>
              <p class="size-footnote" v-else>
                Kui oled väljaspool mõõduvahemikku, võta teenindusega ühendust.
              </p>
              <button
                type="button"
                class="primary-button"
                @click="navigate('checkout')"
              >
                Jätka broneerimisega <span aria-hidden="true">→</span>
              </button>
            </div>
          </div>
        </section>

        <!-- 5. BRONEERIMISVORM -->
        <section
          v-else-if="currentPage === 'checkout'"
          class="content-pad inner-page checkout-page"
        >
          <p class="eyebrow">VIIMANE SAMM</p>
          <h1 class="page-title">Broneeri ratas</h1>
          <form @submit.prevent="confirmBooking">
            <div class="checkout-layout">
              <div class="checkout-contact">
                <h2 class="section-heading">Sinu kontaktandmed</h2>
                <div class="form-fields">
                  <label class="form-field"
                    ><span>Nimi</span
                    ><input
                      v-model.trim="fullName"
                      type="text"
                      name="name"
                      placeholder="Ees- ja perekonnanimi"
                      autocomplete="name"
                      required
                      minlength="2"
                  /></label>
                  <div class="form-two">
                    <label class="form-field"
                      ><span>E-post</span
                      ><input
                        v-model.trim="email"
                        type="email"
                        name="email"
                        placeholder="nimi@naide.ee"
                        autocomplete="email"
                        required
                    /></label>
                    <label class="form-field"
                      ><span>Telefon</span
                      ><input
                        v-model.trim="phone"
                        type="tel"
                        name="phone"
                        placeholder="5xxx xxxx"
                        autocomplete="tel"
                        required
                        minlength="5"
                    /></label>
                  </div>
                </div>
              </div>
              <div class="checkout-review">
                <h2 class="section-heading">Broneeringu kokkuvõte</h2>
                <div class="booking-summary">
                  <div class="booking-row top-row">
                    <div>
                      <small>Sinu valik</small
                      ><strong
                        >{{ selectedBike.name }} · suurus
                        {{ selectedSize }}</strong
                      >
                    </div>
                    <div class="summary-price">
                      {{ euro(selectedBike.price) }}<small>/ kuu</small>
                    </div>
                  </div>
                  <div class="booking-row pickup-row">
                    <div class="pickup-pin">
                      <svg viewBox="0 0 24 24">
                        <path
                          d="M12 21s7-6.1 7-12a7 7 0 1 0-14 0c0 5.9 7 12 7 12Z"
                        />
                        <circle cx="12" cy="9" r="2.5" />
                      </svg>
                    </div>
                    <div>
                      <small>Kättesaamine</small
                      ><strong>Kodar Tartu, Riia 24</strong>
                      <p>Vali sobiv aeg:</p>
                      <div class="pickup-fields">
                        <label
                          ><span>Kuupäev</span
                          ><input
                            v-model="pickupDate"
                            aria-label="Kättesaamise kuupäev"
                            type="date"
                            :min="minDate"
                            required /></label
                        ><label
                          ><span>Kell</span
                          ><input
                            v-model="pickupTime"
                            aria-label="Kättesaamise kellaaeg"
                            type="time"
                            min="09:00"
                            max="19:00"
                            required
                        /></label>
                      </div>
                    </div>
                  </div>
                  <div class="payment-row">
                    <span>Tasumine</span><strong>Kättesaamisel</strong>
                  </div>
                </div>
                <p v-if="formError" class="form-error" role="alert">
                  {{ formError }}
                </p>
                <button class="primary-button" type="submit">
                  Kinnita broneering <span aria-hidden="true">→</span>
                </button>
                <p class="form-note">
                  <span>✓</span> Kontot ega makseandmeid pole vaja
                </p>
              </div>
            </div>
          </form>
        </section>

        <!-- 6. KINNITUS -->
        <section
          v-else-if="currentPage === 'success'"
          class="content-pad inner-page success-page"
        >
          <div class="success-check">
            <svg viewBox="0 0 24 24"><path d="m5 12 4 4L19 6" /></svg>
          </div>
          <p class="eyebrow">BRONEERING KINNITATUD</p>
          <h1 class="page-title">Ratas on broneeritud!</h1>
          <p class="page-subtitle">
            {{ fullName.split(" ")[0] || "Aitäh" }}, sinu näidisbroneering on
            valmis.
          </p>
          <div class="confirmation-number">
            <small>KINNITUSNUMBER</small
            ><strong>{{ confirmationNumber }}</strong>
          </div>
          <div class="booking-summary">
            <div class="booking-row top-row">
              <div>
                <small>Sinu valik</small
                ><strong
                  >{{ selectedBike.name }} · suurus {{ selectedSize }}</strong
                >
              </div>
              <div class="summary-price">
                {{ euro(selectedBike.price) }}<small>/ kuu</small>
              </div>
            </div>
            <div class="booking-row pickup-row">
              <div class="pickup-pin">
                <svg viewBox="0 0 24 24">
                  <path d="M12 21s7-6.1 7-12a7 7 0 1 0-14 0c0 5.9 7 12 7 12Z" />
                  <circle cx="12" cy="9" r="2.5" />
                </svg>
              </div>
              <div>
                <small>Kättesaamine</small><strong>Kodar Tartu, Riia 24</strong>
                <p>{{ dateText(pickupDate) }} kell {{ pickupTime }}</p>
              </div>
            </div>
            <div class="payment-row">
              <span>Tasumine</span><strong>Kättesaamisel</strong>
            </div>
          </div>
          <div class="document-note">
            <span>ID</span>
            <p>
              Võta ratast kätte saades kaasa
              <strong>isikut tõendav dokument.</strong>
            </p>
          </div>
          <p class="demo-notice">
            See on õppeprojekti prototüüp. Päris broneeringut ega kinnituskirja
            ei saadeta.
          </p>
          <button class="outline-button" type="button" @click="reset">
            Tagasi avalehele
          </button>
        </section>
      </main>
    </div>
  </div>
</template>
