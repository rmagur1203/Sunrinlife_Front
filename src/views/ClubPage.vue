<template>

<div class="panel">
    <div class="clud-page">
        <div class="page-content" :class="{'mobile-ui' : isMobileWindow}">
            <div class="club-list-wrap">
                <!-- 학과 선택창 -->
                <div class="major-selecter neu-morphism-card none-dragging">

                    <transition name="current-major-ani">
                        <div class="current-major"
                            v-if="!isShowDivisions"
                            @click="isShowDivisions = true;"
                            :class="{ [division]: true }">
                            <span v-if="division === 'security'">정보보호과</span>
                            <span v-else-if="division === 'software'">소프트웨어과</span>
                            <span v-else-if="division === 'buisness'">IT경영과</span>
                            <span v-else-if="division === 'design'">콘텐츠 디자인과</span>
                            <span v-else-if="division === 'general'">일반 동아리</span>
                            <span v-else-if="division === 'autonomous'">자율 동아리</span>
                        </div>
                    </transition>

                    <transition name="choice-major-ani">
                        <div class="choice-major-wrap none-dragging" v-if="isShowDivisions" @click="isShowDivisions = false">
                            <div class="choice-major security"
                                @click="change('security')">
                                정보
                            </div>
                            <div class="choice-major software"
                                @click="change('software')">
                                솦과
                            </div>
                            <div class="choice-major ceo"
                                @click="change('buisness')">
                                아경
                            </div>
                            <div class="choice-major design"
                                @click="change('design')">
                                콘디
                            </div>
                            <div class="choice-major"
                                @click="change('general')">
                                일반
                            </div>
                            <div class="choice-major security"
                                @click="change('autonomous')">
                                자율
                            </div>
                        </div>
                    </transition>
                </div>

                <!-- 동아리 리스트 -->
                <ul v-if="loadedClubData" class="club-list neu-morphism-card none-dragging">
                    <li
                        v-for="(i, n) in loadedClubData" :key="n"
                        :class="{ 'seleted' : selectIndex == n}"
                        @click="selectIndex = n">
                        <img v-if="i.logo_url != ''" :src="i.logo_url" :alt="`${i.name} 동아리 로고`" class="club-icon">
                        <img v-else src="@/assets/symbol.svg" :alt="`동아리 로고 기본값`" class="club-icon">

                        <div v-if="!isMobileWindow" class="club-name">{{ i.name }}</div>
                    </li>
                </ul>
            </div>

            <ClubCard
                    v-if="!isMobileWindow && loadedClubData && loadedClubData[selectIndex]"
                    :selectClubData="loadedClubData[selectIndex]" />
            <ClubCardMobile
                    v-else-if="isMobileWindow && loadedClubData && loadedClubData[selectIndex]"
                    :selectClubData="loadedClubData[selectIndex]"/>
        </div>
    </div>
</div>
</template>

<script>
import ClubCard from "../components/club/ClubCard.vue"
import ClubCardMobile from "../components/club/ClubCardMobile.vue"



// import { JB, SW, it, de } from "@/components/club/TempClubData"
import { mapState } from 'vuex'
import {
    getClubMajor,
    getClubGeneral,
    getClubAutonomous,} from "@/api"
import store from "@/store"

export default {
    name : "Club Page",
    data(){
        return{
            isShowDivisions: false,
            selectIndex: 0,
        }
    },
    computed :{
        ...mapState(["isMobileWindow", "clubData"]),
        getAuthToken() {
            return this.$store.getters.getAuthToken;
        },
        loadedClubData() {
            return this.clubData[this.division];
        },
        division() {
            return this.$route.query.division || "security";
        }
    },
    watch:{
        getAuthToken(){
            this.loadData();
        },
        $route(v){
            this.loadData(v)
        }
    },
    components : {
        ClubCard,
        ClubCardMobile,
    },
    methods: {
        change(division) {
            this.selectIndex = 0;
            this.$router.push({ path: 'club', query: { division }});
            this.selectIndex = 0;
        },
        loadData(){
            // 이미 있는 데이터가 동아리인 경우는 함수 종료
            if(this.clubData[this.division] !== undefined){
                return
            }


            //동아리 종류와 id메칭
            let clubIdx
            if(this.division === "security"){
                clubIdx = 0
            }
            else if(this.division === "software"){
                clubIdx = 1
            }
            else if(this.division === "buisness"){
                clubIdx = 2
            }
            else if(this.division === "design"){
                clubIdx = 3
            }
            else if(this.division === "general"){
                clubIdx = 4
            }
            else if(this.division === "autonomous"){
                clubIdx = 5
            }

            // 데이터가 없는 동아리인 경우에는 서버 통신
            if(clubIdx <= 3){
                getClubMajor(clubIdx).then((data) => {
                    store.commit("setClubData", { id:  this.division, data });
                });
            }
            else if(clubIdx === 4){
                getClubGeneral().then((data) => {
                    store.commit("setClubData", { id: this.division, data });
                });
            }
            else if(clubIdx === 5){
                getClubAutonomous().then((data) => {
                    store.commit("setClubData", { id: this.division, data });
                });
            }
        }
    },
    mounted(){
        if (this.$store.getters.getAuthToken !== null) this.loadData(this.division);
    }
}
</script>

<!-- 메인 -->
<style scoped>
.page-content {
    display: grid;
    grid-template-columns: auto 1fr;
    gap : 30px;
    margin-top:90px;
}

.club-list-wrap {
    width : 250px;
}

.club-list-wrap > * {
    width : 100%;
}

@media (max-width : 970px) {
    .page-content {
        grid-template-columns : unset;
        grid-template-rows: auto auto;
    }

    .club-list-wrap {
        width : 100%;

        display: flex;
        gap : 24px;
    }

    .club-card .header {
        flex-direction: column;
        align-items: flex-start;
    }

    .club-card .sns-ranker-wrap, .curriculum-wrap {
        padding : 0;
    }
}

@media (max-width : 700px) {
    .club-list-wrap {
        width : 100%;

        display: flex;
        flex-direction: column;
        gap : 24px;
    }
}
</style>

<!-- 학과 선택 -->
<style scoped>
.major-selecter {
    height: 44px;
    margin-bottom : 24px;
    position: relative;
    overflow: hidden;
    font-weight: 700;
}

.major-selecter * {
    cursor: pointer;
}

.major-selecter > div {
    width : 100%;
    height : 100%;
    display: flex;
    position: absolute;
    top : 0px;
    left : 0px;
}

.major-selecter .current-major {
    font-size: 16px;
    font-weight: 700;
    justify-content: center;
    align-items: center;
}

.major-selecter .choice-major-wrap {
    align-items: stretch;
}

.major-selecter .choice-major-wrap .choice-major {
    flex-grow: 1;
    display: flex;
    justify-content: center;
    align-items: center;
}

.major-selecter .choice-major-wrap input {
    display: none;
}

@media (max-width : 970px) {
    .major-selecter {
        height: 52px;
        margin-bottom: 0px;
    }
}


.current-major-ani-enter-from, .current-major-ani-leave-to {
    transform: translateY(-100%);
}

.current-major-ani-enter-active, .current-major-ani-leave-active {
    transition: 300ms;
}

.current-major-ani-enter-to, .current-major-ani-leave-from {
    transform: translateY(0);
}

.choice-major-ani-enter-from, .choice-major-ani-leave-to {
    transform: translateY(100%);
}

.choice-major-ani-enter-active, .choice-major-ani-leave-active {
    transition: 300ms;
}

.choice-major-ani-enter-to, .choice-major-ani-leave-from {
    transform: translateY(0%);
}
</style>

<!-- 동아리 목록 -->
<style scoped>

.club-list {
    padding: 25px 16px;

    display: inline-flex;
    flex-direction: column;
    gap : 4px;
}

.club-list li {
    padding: 8px 26px;
    border-radius: 8px;

    display: inline-flex;
    align-items: center;
    gap : 8px;

    cursor: pointer;
}

.club-list li.seleted, .club-list li:hover {
    background-color: var(--gray1);
}

.club-list .club-icon {
    width: 24px;
    height: 24px;
}

.club-list .club-name{
    font-weight: 500;
    font-size:16px;
}

@media (max-width : 970px) {
    .club-list {
        height: 52px;

        padding: 8px;

        flex-direction: row;
        justify-content: space-around;
        gap : 0;
    }

    .club-list li {
        flex : 1;

        padding : 0px;

        display: flex;
        justify-content: center;
        gap : 0px;
    }
}
</style>