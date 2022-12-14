<script setup>
import { ref, reactive } from "vue";

const props = defineProps({
  img: String,
  ratingsTotal: Number,
  rating: Number,
  name: String,
  placeId: String,
  getPlaceDetail: Function,
});

const detail = reactive({
  gmapUrl: "",
  webSiteUrl: "",
  openingHours: {
    openNow: false,
    weekdayText: [],
  },
  phoneNumber: "",
  photos: [""],
  reviews: [],
});

const isVisibleDetail = ref(false);
const isVisibleOpeningHoursWeeklyText = ref(false);

const onClickDetail = async (placeId) => {
  isVisibleDetail.value = true;
  const fetchedDetail = await props.getPlaceDetail(placeId);
  detail.gmapUrl = fetchedDetail.url;
  detail.webSiteUrl = fetchedDetail.website;
  detail.openingHours = {
    openNow: fetchedDetail.opening_hours
      ? fetchedDetail.opening_hours.isOpen()
      : false,
    weekdayText: fetchedDetail.opening_hours
      ? fetchedDetail.opening_hours.weekday_text
      : [],
  };
  detail.phoneNumber = fetchedDetail.formatted_phone_number;
  detail.photos = fetchedDetail.photos;
  detail.reviews = fetchedDetail.reviews;
};

const onclickCloseDetail = () => {
  isVisibleDetail.value = false;
};

const toggleWeeklyText = () => {
  isVisibleOpeningHoursWeeklyText.value =
    !isVisibleOpeningHoursWeeklyText.value;
};
</script>

<template>
  <div class="p-2 flex flex-col gap-y-2 bg-white w-full">
    <div v-if="props.img">
      <img :src="props.img" alt="" class="aspect-[4/3] object-cover rounded" />
    </div>
    <div class="flex items-center gap-x-2">
      <p class="font-bold">
        <span class="text-xl text-secondary">{{
          props.ratingsTotal || 0
        }}</span>
        件
      </p>
      <p v-if="props.rating" class="text-xs">
        <span class="text-accent">★</span>{{ props.rating }}
      </p>
    </div>
    <p class="font-bold text-2xl">
      {{ props.name }}
    </p>
    <button
      v-if="!isVisibleDetail"
      class="flex items-center gap-x-2 m-auto"
      @click="onClickDetail(props.placeId)"
    >
      <div>もっと見る</div>
      <img src="assets/icon-down-arrow.svg" alt="" class="w-4" />
    </button>
    <div v-else class="flex flex-col gap-y-2">
      <div class="border-t-2 border-base-100"></div>
      <div class="flex flex-col gap-y-2">
        <div class="flex items-center gap-x-4">
          <img src="assets/icon-gmap.png" alt="" class="w-5" />
          <div class="text-base">
            <a
              :href="detail.gmapUrl"
              target="_blank"
              rel="noopener noreferrer"
              class="link"
            >
              Google Mapで見る
            </a>
          </div>
        </div>
        <div v-if="detail.webSiteUrl" class="flex items-center gap-x-4">
          <img src="assets/icon-website.png" alt="" class="w-5" />
          <div
            class="text-base overflow-hidden text-ellipsis whitespace-normal w-full"
          >
            <a
              :href="detail.webSiteUrl"
              target="_blank"
              rel="noopener noreferrer"
              class="link"
              >{{ detail.webSiteUrl }}</a
            >
          </div>
        </div>
        <div
          v-if="detail.openingHours.weekdayText[0]"
          class="flex items-center gap-x-4"
        >
          <img src="assets/icon-watch.png" alt="" class="w-5" />
          <button
            v-if="detail.openingHours.openNow"
            class="flex items-center gap-x-2"
            @click="toggleWeeklyText()"
          >
            <div class="text-base text-success-content">営業中</div>
            <img
              :src="
                isVisibleOpeningHoursWeeklyText
                  ? 'assets/icon-up-arrow.svg'
                  : 'assets/icon-down-arrow.svg'
              "
              alt=""
              class="w-4"
            />
          </button>
          <div v-else class="flex items-center gap-x-2 text-error text-base">
            営業時間外
          </div>
        </div>
        <ul
          v-if="isVisibleOpeningHoursWeeklyText"
          class="ml-9 text-sm overflow-auto whitespace-nowrap flex flex-col gap-y-0.5"
        >
          <li
            v-for="(text, index) in detail.openingHours.weekdayText"
            :key="index"
          >
            {{ text }}
          </li>
        </ul>
        <div v-if="detail.phoneNumber" class="flex items-center gap-x-4">
          <img src="assets/icon-phone.png" alt="" class="w-5" />
          <div class="text-base">
            <a href="tel:090-1234-5678" class="link">{{
              detail.phoneNumber
            }}</a>
          </div>
        </div>
      </div>
      <div class="border-t-2 border-base-100"></div>
      <div v-if="detail.photos" class="carousel">
        <div
          v-for="(photo, index) in detail.photos"
          :key="index"
          class="carousel-item w-[120px] mr-2"
        >
          <img
            :src="photo ? photo.getUrl() : ''"
            alt=""
            class="aspect-[4/3] object-cover rounded"
          />
        </div>
      </div>
      <div class="border-t-2 border-base-100"></div>
      <div class="flex flex-col gap-y-2">
        <div
          v-for="(review, index) in detail.reviews"
          :key="index"
          class="flex flex-col gap-y-1 text-xs"
        >
          <div class="flex items-center gap-x-2">
            <img :src="review.profile_photo_url" alt="" width="24" />
            <div>{{ review.author_name }}</div>
          </div>
          <div class="flex items-center gap-x-2">
            <div>{{ review.relative_time_description }}</div>
            <div><span class="text-accent">★</span>{{ review.rating }}</div>
          </div>
          <div class="text-sm">
            {{ review.text }}
          </div>
        </div>
      </div>
      <button
        class="flex items-center gap-x-2 mx-auto mt-2"
        @click="onclickCloseDetail()"
      >
        <div>閉じる</div>
        <img src="assets/icon-up-arrow.svg" alt="" class="w-4" />
      </button>
    </div>
  </div>
</template>
