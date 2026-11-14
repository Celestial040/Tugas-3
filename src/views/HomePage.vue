<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonButton, IonIcon, IonList, IonItem } from '@ionic/vue';
import axios from 'axios';

interface Coin {
  id: string;
  rank: number;
  name: string;
  price_usd: string;
}

const cryptoList = ref<Coin[]>([]);
const lastUpdated = ref<string>('Loading...');

const headerDate = ref<string>('');
const headerTime = ref<string>('');

let timer: number | null = null;

const updateHeaderClock = () => {
  const now = new Date();
  const optionsDate: Intl.DateTimeFormatOptions = {
    weekday: 'long',
    day: '2-digit',
    month: '2-digit',
    year: 'numeric',
    timeZone: 'UTC'
  };
  headerDate.value = now.toLocaleDateString('en-GB', optionsDate).replace(/\//g, '-');

  const hours = String(now.getUTCHours()).padStart(2, '0');
  const minutes = String(now.getUTCMinutes()).padStart(2, '0');
  const seconds = String(now.getUTCSeconds()).padStart(2, '0');
  headerTime.value = `${hours}:${minutes}:${seconds}`;
};

const formatUtcTimestamp = (unixTime: number) => {
  const date = new Date(unixTime * 1000);
  const day = String(date.getUTCDate()).padStart(2, '0');
  const month = String(date.getUTCMonth() + 1).padStart(2, '0');
  const year = date.getUTCFullYear();
  const hours = String(date.getUTCHours()).padStart(2, '0');
  const minutes = String(date.getUTCMinutes()).padStart(2, '0');
  const seconds = String(date.getUTCSeconds()).padStart(2, '0');
  return `${day}-${month}-${year} ${hours}:${minutes}:${seconds}`;
};

const fetchCryptoData = async () => {
  try {
    const response = await axios.get('https://api.coinlore.net/api/tickers/');
    cryptoList.value = response.data.data.slice(0, 10);

    if (response.data.info && response.data.info.time) {
      lastUpdated.value = formatUtcTimestamp(response.data.info.time);
    } else {
      const now = new Date();
      lastUpdated.value = `${String(now.getUTCDate()).padStart(2, '0')}-${String(now.getUTCMonth() + 1).padStart(2, '0')}-${now.getUTCFullYear()} ${String(now.getUTCHours()).padStart(2, '0')}:${String(now.getUTCMinutes()).padStart(2, '0')}:${String(now.getUTCSeconds()).padStart(2, '0')}`;
    }
  } catch (error) {
    console.error(error);
  }
};

onMounted(() => {
  updateHeaderClock();
  timer = window.setInterval(updateHeaderClock, 1000);
  fetchCryptoData();
});

onUnmounted(() => {
  if (timer) clearInterval(timer);
});
</script>

<template>
  <ion-page>
    <ion-header>
      <ion-toolbar mode="md" class="bg-gray-900 border-solid border-b border-gray-400">
          <ion-title class="w-4/5 pl-6 text-base font-semibold">Crypto-Kun</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
        <div class="w-full flex flex-col items-center justify-center">
          <div class="flex w-11/12 justify-between items-center px-2.5 py-4">
            <div class="flex flex-col text-xs">
              <p>{{ headerDate }}</p>
              <p>{{ headerTime }}</p>
            </div>
            <ion-button fill="outline" color="medium" size="default" @click="fetchCryptoData">
              <ion-icon color="light">
                <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32"
                fill="white" viewBox="0 0 24 24" >
                <path d="M19.07 4.93a9.9 9.9 0 0 0-3.18-2.14 10.12 10.12 0 0 0-7.79 0c-1.19.5-2.26 1.23-3.18 2.14S3.28 6.92 2.78 8.11A9.95 9.95 0 0 0 1.99 12h2c0-1.08.21-2.13.63-3.11.4-.95.98-1.81 1.72-2.54.73-.74 1.59-1.31 2.54-1.71 1.97-.83 4.26-.83 6.23 0 .95.4 1.81.98 2.54 1.72.17.17.33.34.48.52L16 9.01h6V3l-2.45 2.45c-.15-.18-.31-.36-.48-.52m.3 10.18c-.4.95-.98 1.81-1.72 2.54-.73.74-1.59 1.31-2.54 1.71-1.97.83-4.26.83-6.23 0-.95-.4-1.81-.98-2.54-1.72-.17-.17-.33-.34-.48-.52l2.13-2.13H2v6l2.45-2.45c.15.18.31.36.48.52.92.92 1.99 1.64 3.18 2.14 1.23.52 2.54.79 3.89.79s2.66-.26 3.89-.79c1.19-.5 2.26-1.23 3.18-2.14s1.64-1.99 2.14-3.18c.52-1.23.79-2.54.79-3.89h-2c0 1.08-.21 2.13-.63 3.11Z"></path>
                </svg>
              </ion-icon>
            </ion-button>
          </div>

          <div class="w-full px-4">
            <div class="flex items-center gap-8 justify-between w-full px-4 py-2 text-sm font-light text-center">
              <p class="block w-2/12">Rank</p>
              <p class="block w-7/12">Crypto Name</p>
              <p class="block w-3/12">USD</p>
            </div>
            <ion-list>
                <ion-item v-for="coin in cryptoList" :key="coin.id">
                  <div class="flex flex-col w-full py-2 gap-4 items-center">
                    <div class="flex items-center gap-8 justify-between w-full text-sm font-light text-center pt-4">
                      <p class="block w-2/12">{{ coin.rank }}</p>
                      <p class="block w-7/12">{{ coin.name }}</p>
                      <p class="block w-3/12">{{ coin.price_usd }}</p>
                    </div>
                    <p class="text-xs font-extralight text-gray-400 mr-6">{{ lastUpdated }}</p>
                  </div>
                </ion-item>
              </ion-list>
          </div>
        </div>
      </ion-content>
  </ion-page>
</template>
