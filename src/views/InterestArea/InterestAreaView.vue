<script setup>
import Footer from "@/components/Footer.vue";
import { ref } from "vue";
import axios from "axios";
import { useAuthStore } from "@/stores/auth";
import boardApi from "@/api/boardApi";
import { useRouter } from "vue-router";

const movedDongCode = ref("");
const movedAptCode = ref("");
const router = useRouter();

const pickedInterestArea = ref("");
const pickedDongCode = ref(""); //등록할 때
const searchInput = ref("");
const searchedDongCodes = ref([]);
var cache = "";
const checkInput = (event) => {
  searchInput.value = event.target.value;
  // timer(beforeInput);
  //console.log(searchInput.value);
  loadData(searchInput.value);
};

const loadData = async (input) => {
  const url = `http://localhost/homeis/map/dongCodes/${input}`;
  // 매개변수 input 값에 따라 서버에서 해당 검색어와 연관된 추천검색어가 담긴 데이터가 불러와진다.

  if (cache === url)
    return; // 이전에 부른 데이터랑 다를 때만 fetch로 데이터를 새로 불러온다.
  else {
    cache = url;
    const { data } = await axios.get(url);
    searchedDongCodes.value = data;
    console.log("동 들 = ", searchedDongCodes);
  }
};

const isEmptyResult = () => {
  if (
    searchedDongCodes.value == null ||
    searchInput.value == "" ||
    searchedDongCodes.value.length == 0
  ) {
    return true;
  }
  return false;
};

const hideFlag = ref(true);

const showContainer = () => {
  hideFlag.value = true;
  pickFlag.value = false;
};

const authStore = useAuthStore();
const InterestAreaList = ref([]);
const houseInfoList = ref([]);
const dealInfoList = ref([]);
const pickFlag = ref(false);

const getInterestAreaList = async () => {
  try {
    const { data } = await boardApi.get(
      "/user/interest-area/" + authStore.user.id
    );
    InterestAreaList.value = data;
    console.log(data);
  } catch (error) {
    if (error.response.status == 404) {
      InterestAreaList.value = [];
      houseInfoList.value = [];
    }
  }
};
getInterestAreaList();

const isEmptyAreaList = () => {
  if (InterestAreaList.value == null || InterestAreaList.value.length == 0)
    return true;
  return false;
};

const getDealInfoList = async (aptCode) => {
  const { data } = await boardApi.get("/map/apartDealInfo/" + aptCode);
  dealInfoList.value = data;
};

const isEmptyHouseInfoList = () => {
  //console.log("EERER = ", houseInfoList.value);
  if (houseInfoList.value == null || houseInfoList.value.length == 0) {
    return true;
  }
  return false;
};
const getHouseInfoList = async (dongCode) => {
  movedDongCode.value = dongCode;
  const { data } = await axios.get(
    "http://localhost:80/homeis/map/houseInfo/" + dongCode
  );
  houseInfoList.value = data;
};

const getFourHouseInfoList = () => {
  if (isEmptyHouseInfoList()) return [];
  const fourHouseInfoList = [];
  for (let i = 0; i < houseInfoList.value.length; i++) {
    if (i == 4) break;
    fourHouseInfoList.push(houseInfoList.value[i]);
  }
  return fourHouseInfoList;
};

const deleteInterestArea = async (areaId) => {
  if (!confirm("해당 지역을 삭제하시겠습니까?")) return;
  await boardApi.delete("/user/interest-area/" + areaId);
  alert("정상적으로 해당 지역이 삭제되었습니다.");
  getInterestAreaList();
};

const modalCheck = ref(false);
const modalOpen = () => {
  if (InterestAreaList.value.length == 4) {
    alert("관심지역은 최대 4개까지 가능합니다.");
    return;
  }
  console.log(modalCheck.value);
  modalCheck.value = !modalCheck.value;
};

const pickArea = async (dongCode, dongName) => {
  pickedInterestArea.value = dongName;
  pickedDongCode.value = dongCode;
  hideFlag.value = false;
  pickFlag.value = true;
};

const insertInterestArea = async () => {
  if (!confirm("정말 해당 지역을 등록하시겠습니까?")) return;

  for (let i = 0; i < InterestAreaList.value.length; i++) {
    if (pickedDongCode.value == InterestAreaList.value[i].dongCode) {
      alert("이미 등록된 지역은 등록할 수 없습니다!");
      pickedDongCode.value = "";
      pickedInterestArea.value = "";
      return;
    }
  }

  await boardApi.post("/user/interest-area", {
    dongCode: pickedDongCode.value,
  });

  alert("정상적으로 등록되었습니다.");
  getInterestAreaList();
  modalCheck.value = false;
  pickedDongCode.value = "";
  pickedInterestArea.value = "";
};

const moveMap = (aptCode) => {
  movedAptCode.value = aptCode;
  const url = `/map/${movedDongCode.value}/${movedAptCode.value}`;
  router.push({
    name: "Map",
    params: { dongCode: movedDongCode.value, aptCode: movedAptCode.value },
  });
};

const moveDong = () => {
  const url = `/map/${movedDongCode.value}/0`;
  router.push({
    name: "Map",
    params: { dongCode: movedDongCode.value, aptCode: 0 },
  });
};

const isEmptyPickedDongCode = () => {
  return movedDongCode.value == "";
};
</script>

<template>
  <div id="interest-main">
    <div class="modal-wrap" v-show="modalCheck" @click="modalOpen()">
      <div class="modal-container" @click.stop="">
        <div class="modal-btn">
          <div class="model-title">관심지역 등록</div>
          <button class="modal-cancel-btn" @click="modalOpen">
            <i class="fa-solid fa-xmark" style="color: #ff0000"></i>
          </button>
        </div>
        <div class="searchBox">
          <input
            type="text"
            class="searchTxt"
            name=""
            placeholder="대한민국에 있는 모든 동을 검색해보세요."
            @input="checkInput"
            v-model="inputDongTmp"
            @click="showContainer()"
          />
          <div></div>
          <button class="searchBtn" type="submit">
            <i class="fa-solid fa-magnifying-glass"></i>
          </button>
        </div>
        <container class="rel_search" v-show="!pickFlag">
          <ul class="pop_rel_keywords">
            <li
              class="li-bottom"
              @click="pickArea(dong.dongCode, dong.dongName)"
              v-show="hideFlag"
              v-for="dong in searchedDongCodes"
              :key="dong.dongCode"
            >
              {{ dong.sidoName + " " + dong.gugunName + " " + dong.dongName }}
            </li>
          </ul>
        </container>
        <div class="picked-area" v-show="pickedInterestArea != ''">
          <div>{{ pickedInterestArea }}</div>
        </div>
        <div class="regist-div">
          <button
            type="button"
            @click="insertInterestArea()"
            class="area-regist-btn"
            v-show="pickFlag"
          >
            등록
          </button>
        </div>
      </div>
    </div>

    <div id="interest-main-content">
      <div id="interest-main-title">관심지역</div>
      <div id="interest-main-smallTitle">
        ※ 관심지역은 최대 4개까지 가능합니다
      </div>
      <div id="interest-list">
        <div id="addButton" @click="modalOpen()">
          <i class="fa-solid fa-plus"></i>
        </div>
        <div id="interest-empty-list" v-show="isEmptyAreaList()">
          <div id="interest-empty-msg">
            좌측 &nbsp;<i class="fa-solid fa-plus"></i>&nbsp; 버튼으로
            관심지역을 추가해보세요!
          </div>
        </div>
        <div id="interest-list-block" v-show="!isEmptyAreaList()">
          <div
            v-for="areaInfo in InterestAreaList"
            id="dong-block"
            @click="getHouseInfoList(areaInfo.dongCode)"
          >
            {{ areaInfo.dongName }}
            <span class="close" @click="deleteInterestArea(areaInfo.id)">
              <i class="fa-solid fa-xmark" style="color: #ff0000"></i>
            </span>
          </div>
        </div>
      </div>
      <div id="interest-deal">
        <div
          id="house-info-box"
          v-for="houseInfo in getFourHouseInfoList()"
          v-show="!isEmptyHouseInfoList()"
          @click="moveMap(houseInfo.aptCode)"
        >
          <div id="house-img"></div>
          <div id="house-info">
            <div>{{ houseInfo.apartmentName }}</div>
            <div>건축년도 : {{ houseInfo.buildYear }}</div>
            <div>지번 : {{ houseInfo.dong }} {{ houseInfo.jibun }}</div>
          </div>
        </div>
        <div
          id="empty-deal-title"
          v-show="isEmptyHouseInfoList() && !isEmptyPickedDongCode()"
        >
          <p>거래이력이 없는 동입니다.</p>
          <p>
            거래가 될 때까지 조금만 기다려주세요
            <i class="fa-solid fa-face-sad-tear" style="color: #ffd43b"></i>
          </p>
        </div>

        <div
          id="extend_search"
          @click="moveDong()"
          v-show="!isEmptyHouseInfoList()"
        >
          <div>더 많은 정보를 보시려면 이 곳을 누르세요!!</div>
        </div>
      </div>
    </div>
  </div>
  <Footer />
</template>

<style scoped>
.picked-area {
  margin-left: auto;
  margin-right: auto;
  text-align: center;
  margin-top: 20px;
  width: 150px;
  padding: 10px 0;
  animation-duration: 1s;
  animation-name: rainbow-b;
  animation-iteration-count: infinite;
  font-weight: 800;
  font-size: 1.4rem;
  border-radius: 10px;
}
.regist-div {
  margin-top: 20px;
  text-align: center;
}
.area-regist-btn {
  background-color: #f9f9f9;
  cursor: pointer;
  font-size: 24px;
  font-weight: 600;
  padding: 5px 10px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
.model-title {
  font-size: 30px;
  font-weight: 600;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
}
.rel_search {
  z-index: 100;
  height: 50%;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  border-radius: 12px;
  overflow-y: auto;
  overflow-x: hidden;
}

.li-bottom {
  border-bottom: 1px solid gray;
}

.li-bottom:hover {
  background-color: rgb(88, 153, 214);
}

.pop_rel_keywords {
  list-style: none;
  width: 500px;
  background-color: white;
  border-radius: 12px;
  padding: 10px;
}

.pop_rel_keywords > li {
  /* JS에서 동적으로 li를 생성할 때 적용될 스타일*/
  line-height: 250%;
}

.searchBox {
  height: 50px;
  width: 500px;
  background: white;
  border-radius: 40px;
  padding: 10px;
  border: 1px solid #f9deec;
  display: flex;
  justify-content: center;
  z-index: 10;
}
.searchTxt {
  border: none;
  background: none;
  outline: none;
  float: left;
  padding: 0px;
  color: #797979;
  font-size: 20px;
  line-height: 30px;
  width: 400px;
}

.searchBtn {
  color: #f9deec;
  float: right;
  width: 40px;
  height: 100%;
  border-radius: 50%;
  background: white;
  border: none;
  font-size: 24px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-btn {
  display: flex;
  justify-content: space-between;
  margin-bottom: 30px;
  margin-right: 10px;

  .modal-cancel-btn {
    position: absolute;
    background-color: #fff;
    font-size: 2rem;
    border-style: none;
    cursor: pointer;
    top: 1vh;
    right: 1vw;
  }
}

.close {
  position: absolute;
  font-size: 1.2rem;
  right: 0.3vw;
  top: 0.3vh;
}
.modal-wrap {
  z-index: 100;
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.4);
}
/* modal or popup */
.modal-container {
  position: relative;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 550px;
  height: 60%;
  background: #fff;
  border-radius: 10px;
  padding: 30px;
  box-sizing: border-box;
}

#extend_search {
  margin-top: 7vh;
  text-align: center;
  font-size: 2.3rem;
  font-weight: 600;
}

#house-info-box {
  display: flex;
  height: 22%;
  background-color: white;
  margin-bottom: 3vh;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  #house-img {
    width: 30%;
    margin: 3%;
    background-image: url("/src/assets/img/sample_apart.jpg");
    background-repeat: no-repeat;
    background-position: center center;
    background-size: cover;
  }
  #house-info {
    width: 70%;
    margin: 3%;
    font-weight: 600;
    border-radius: 10px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    div {
      margin-left: 10px;
      margin-bottom: 10px;
    }
  }
}
#house-info-box:hover {
  border: 2px solid black;
}
#interest-main {
  width: 99vw;
  height: 150vh;
  padding-top: 15vh;
  padding-left: 20vw;
  padding-right: 20vw;
  padding-bottom: 10vh;
}
#interest-main-content {
  background-color: #f9f9f9;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  width: 100%;
  height: 100%;
  padding: 5vh;
  display: flex;
  flex-direction: column;
  gap: 2vh;
  #interest-main-title {
    width: 100%;
    height: 7%;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 2.3rem;
  }
  #interest-main-smallTitle {
    height: 3%;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 10px;
  }
  #interest-list {
    display: flex;
    width: 100%;
    height: 10%;
    #addButton {
      width: 10%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      i {
        font-size: 2rem;
        border-radius: 50%;
        cursor: pointer;
      }
    }
    #interest-empty-list {
      width: 90%;
      height: 100%;
      font-size: 1.5rem;
      display: flex;
      justify-content: center;
      align-items: center;
      padding-top: 3vh;
      padding-bottom: 3vh;
      padding-left: 2vw;
      i {
        border: 3px solid black;
        border-radius: 50%;
      }
      #interest-empty-msg {
        height: 100%;
        width: 100%;
        background-color: #dee2ef;
        display: flex;
        justify-content: center;
        align-items: center;
        border-radius: 10px;
      }
    }
    #interest-list-block {
      width: 90%;
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      gap: 3vw;
      #dong-block {
        width: 180px;
        height: 60px;
        border: 1px solid white;
        font-weight: 800;
        font-size: 1.4rem;
        text-align: center;
        background-color: white;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        display: flex;
        justify-content: center;
        align-items: center;
        position: relative;
      }
      #dong-block:hover {
        border-color: black;
        background-color: black;
        color: white;
      }
    }
  }
}

#interest-deal {
  width: 100%;
  height: 75%;
  padding-top: 4vh;
  padding-left: 5vw;
  padding-right: 5vw;
  padding-bottom: 4vw;
}

#interest-intro-main {
  border: 1px solid black;
}

#empty-deal-title {
  font-size: 30px;
  font-weight: 600;
  text-align: center;
}

.interest-intro {
  width: 800px;
  height: 100px;
  border: 1px solid black;
  font-size: 30px;
  text-align: center;
}
</style>
