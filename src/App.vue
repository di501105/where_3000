<template>
  <div id="app">
    <div class="treble">
      <div class="treble__sidebar" :class="isShowSidebar ? 'active' : ''">
        <h1 class="sidebar__title">三倍券地圖</h1>
        <div class="sidebar__select">
          <el-select class="select__item" v-model="select.city" placeholder="請選擇縣市"
          @change="selectCity()">
            <el-option v-for="city in cityCountyData"
            :label="city.CityName" :value="city.CityName" :key="city.CityEngName"></el-option>
          </el-select>
          <el-select class="select__item" v-model="select.area" placeholder="請選擇鄉鎮區"
          :disabled="select.city === ''" @change="selectArea()">
            <el-option v-for="area in areaList"
            :label="area.AreaName" :value="area.AreaName" :key="area.AreaEngName"></el-option>
          </el-select>
        </div>
        <div class="sidebar__list">
          <div class="list__item" v-for="post in selectPost" :key="post.storeCd"
          @click="switchSidebar(); panTo(post);">
            <h2 class="item__title">{{ post.storeNm }}</h2>
            <p class="item__text">{{ post.addr }}</p>
            <p class="item__text">{{ post.tel }}</p>
            <p class="item__text" v-html="post.busiTime"></p>
            <div class="item__total"
            :class="[post.total > 0 ? 'item__total--inStock' : 'item__total--empty']">
              <p class="total__text">數量</p>
              <p class="total__text">{{ post.total }}</p>
            </div>
          </div>
        </div>
      </div>
      <div class="treble__map" @click="closeSidebar()">
        <div id="map"></div>
      </div>
      <div class="treble__float" @click="switchSidebar()">
        <div class="arrow__icon"></div>
      </div>
    </div>
  </div>
</template>

<script>
import L from 'leaflet';
import cityCountyData from '@/assets/CityCountyData.json';

export default {
  name: 'App',
  data() {
    return {
      cityCountyData,
      postList: [],
      areaList: [],
      selectPost: [],
      osmMap: {},
      select: {
        city: '',
        area: '',
      },
      isShowSidebar: true,
    };
  },
  created() {
    this.getPostList();
  },
  mounted() {
    this.createMap();
  },
  methods: {
    async getPostList() {
      const url = 'https://3000.gov.tw/hpgapi-openmap/api/getPostData';
      try {
        const res = await this.axios.get(url);
        this.postList = res.data;
      } catch (e) {
        console.log(e);
      }
    },
    createMap() {
      this.osmMap = L.map('map', {
        center: [25.03, 121.55],
        zoom: 15,
      });
      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '<a target="_blank" href="https://www.openstreetmap.org/">© OpenStreetMap 貢獻者</a>',
        maxZoom: 18,
      }).addTo(this.osmMap);
    },
    selectCity() {
      this.removeMapMarker();
      const selectCity = this.cityCountyData.find((city) => city.CityName === this.select.city);
      this.areaList = selectCity.AreaList;
      this.select.area = '';
      this.selectPost = this.postList.filter((city) => city.hsnNm === this.select.city);
      this.selectPost.map((post) => this.addMapMarker(post.longitude, post.latitude, post));
      this.panTo(this.selectPost[0]);
    },
    selectArea() {
      this.removeMapMarker();
      this.selectPost = this.postList.filter((city) => city.hsnNm === this.select.city)
        .filter((area) => area.townNm === this.select.area);
      this.selectPost.map((post) => this.addMapMarker(post.longitude, post.latitude, post));
      this.panTo(this.selectPost[0]);
    },
    addMapMarker(x, y, post) {
      L.marker([y, x]).addTo(this.osmMap).bindPopup(`
            <h2 class="item__title">${post.storeNm}</h2>
            <a href="https://www.google.com.tw/maps/place/${post.addr}" target="_blank" class="popup__text">${post.addr}</a>
            <p class="popup__text">${post.tel}</p>
            <p class="popup__text">${post.busiTime}</p>
            <div class="popup__total ${post.total > 0 ? 'popup__total--inStock' : 'popup__total--empty'}">
              <p class="total__text">數量</p>
              <p class="total__text">${post.total}</p>
            </div>`);
    },
    removeMapMarker() {
      this.osmMap.eachLayer((layer) => {
        if (layer instanceof L.Marker) {
          this.osmMap.removeLayer(layer);
        }
      });
    },
    panTo(post) {
      this.osmMap.panTo(new L.LatLng(post.latitude, post.longitude));
      L.marker([post.latitude, post.longitude]).addTo(this.osmMap).bindPopup(`
            <h2 class="popup__title">${post.storeNm}</h2>
            <a href="https://www.google.com.tw/maps/place/${post.addr}" target="_blank" class="popup__text">${post.addr}</a>
            <p class="popup__text">${post.tel}</p>
            <p class="popup__text">${post.busiTime}</p>
            <div class="popup__total ${post.total > 0 ? 'popup__total--inStock' : 'popup__total--empty'}">
              <p class="total__text">數量</p>
              <p class="total__text">${post.total}</p>
            </div>`).openPopup();
    },
    switchSidebar() {
      this.isShowSidebar = !this.isShowSidebar;
    },
    closeSidebar() {
      this.isShowSidebar = false;
    },
  },
};
</script>

<style lang="scss">
* {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
  font-family: "微軟正黑體", "Microsoft JhengHei", Helvetica, arial, sans-serif
}
.treble {
  width: 100%;
  display: flex;
  .treble__sidebar {
    width: 20%;
    display: flex;
    flex-direction: column;
    align-items: center;
    background: #73C0D8;
    padding-top: 20px;
    .sidebar__title {
      color: #fff;
    }
    .sidebar__select {
      border-bottom: 1px solid #D9D9D9;
      padding: 10px 10px 25px;
      .select__item {
        width: 100%;
        &:first-of-type {
          margin-bottom: 10px;
        }
      }
    }
    .sidebar__list {
      background: #fff;
      width: 100%;
      height: 80vh;
      padding: 0 10px;
      overflow-y: auto;
      .list__item {
        padding: 10px;
        border-bottom: 1px solid #D9D9D9;
        .item__title {
          color: #333333;
          font-weight: Bold;
          font-size: 16px;
          margin-bottom: 5px;
        }
        .item__text {
          color: #666666;
          font-size: 12px;
          margin-bottom: 5px;
        }
        .item__total {
          display: flex;
          justify-content: space-between;
          padding: 5px 10px;
          border-radius: 50px;
          &--inStock {
            background: #FFA573;
          }
          &--empty {
            background: #aaaaaa;
          }
          .total__text {
            color: #ffffff;
            font-size: 14px;
            letter-spacing: 1px;
          }
        }
      }
    }
  }
  .treble__map {
    width: 80%;
    #map {
      height: 100vh;
    }
  }
  .leaflet-popup-content {
    .popup__title {
      color: #333333;
      font-weight: Bold;
      font-size: 16px;
      margin-bottom: 5px;
    }
    .popup__text {
      color: #666666;
      font-size: 12px;
      margin-bottom: 5px;
    }
    .popup__total {
      display: flex;
      justify-content: space-between;
      padding: 5px 10px;
      border-radius: 50px;
      &--inStock {
        background: #FFA573;
      }
      &--empty {
        background: #aaaaaa;
      }
      .total__text {
        color: #ffffff;
        font-size: 14px;
        letter-spacing: 1px;
        margin: 0;
      }
    }
  }
}
@media screen and (max-width: 500px){
  .treble {
    display: block;
    position: relative;
    .treble__sidebar {
      position: absolute;
      top: 0;
      transform: translateX(-100%);
      transition: all .3s;
      width: 80%;
      z-index: 999;
      height: 100vh;
      &.active {
        transform: translateX(0);
      }
    }
    .treble__map {
      position: relative;
      width: 100%;
      #map {
        height: 100vh;
      }
    }
    .treble__float {
      position: absolute;
      z-index: 990;
      width: 30px;
      height: 90px;
      border-radius: 0 20px 20px 0;
      background: #FFA573;
      left: 0;
      top: 20%;
      display: flex;
      justify-content: center;
      align-items: center;
      .arrow__icon {
        width: 20px;
        height: 20px;
        border-left: 2px solid white;
        border-bottom: 2px solid white;
        transform: rotate(225deg) skew(20deg, 20deg);
      }
    }
    .leaflet-control-container {
      .leaflet-control {
        display: none;
      }
    }
  }
}

</style>
