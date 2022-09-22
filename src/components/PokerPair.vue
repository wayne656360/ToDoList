<template>
    <h1 class="pokerh1">撲克牌翻牌配對遊戲</h1>
    <section id="pokerapp">

        <div class="cardtable">
            <div class="card" @click="transCard(card)" v-for="card in remderCards" :key="card.id">
                <div class="front" :class="{transfront: card.ispair}" :style="{ color: card.cardColor }">
                    <span>{{ card.cardText }}</span>
                </div>
                <div class="back" :class="{transback: card.ispair}"></div>
            </div>
            <div class="restart_wrap" v-if="count === 8">
                <p>恭喜挑戰遊戲成功!!!</p>
                <button class="replay" @click="restartGame">再玩一局</button>
            </div>
        </div>


    </section>
</template>
<script setup>
import { ref, onMounted, computed, watch, inject } from 'vue' 
// 卡片花色資料建立
const cardlist = ref([
    { id: 1, cardText: '♠1', cardColor: 'black', ispair: false, sortIdx: 0 },
    { id: 5, cardText: '♠1', cardColor: 'black', ispair: false, sortIdx: 0 },
    { id: 7, cardText: '♥2', cardColor: 'red', ispair: false, sortIdx: 0 },
    { id: 4, cardText: '♥2', cardColor: 'red', ispair: false, sortIdx: 0 },
    { id: 2, cardText: '♦7', cardColor: 'black', ispair: false, sortIdx: 0 },
    { id: 14, cardText: '♦7', cardColor: 'black', ispair: false, sortIdx: 0 },
    { id: 12, cardText: '♣7', cardColor: 'red', ispair: false, sortIdx: 0 },
    { id: 8, cardText: '♣7', cardColor: 'red', ispair: false, sortIdx: 0 },
    { id: 9, cardText: '♠10', cardColor: 'black', ispair: false, sortIdx: 0 },
    { id: 3, cardText: '♠10', cardColor: 'black', ispair: false, sortIdx: 0 },
    { id: 16, cardText: '♥J', cardColor: 'red', ispair: false, sortIdx: 0 },
    { id: 11, cardText: '♥J', cardColor: 'red', ispair: false, sortIdx: 0 },
    { id: 10, cardText: '♦Q', cardColor: 'black', ispair: false, sortIdx: 0 },
    { id: 15, cardText: '♦Q', cardColor: 'black', ispair: false, sortIdx: 0 },
    { id: 6, cardText: '♣K', cardColor: 'red', ispair: false, sortIdx: 0 },
    { id: 13, cardText: '♣K', cardColor: 'red', ispair: false, sortIdx: 0 },
])
// 產生隨機順序
let randomIndexList = [];
function randomIdx() {
    for (let i = 0; i < 16; i++) {
        let num = parseInt(Math.random() * 16);
        if (!randomIndexList.includes(num)) {
            randomIndexList.push(num);
        }
    }
    for (let i = 0; i < 16; i++) {
        if (!randomIndexList.includes(i)) {
            if (i % 2 === 0) {
                randomIndexList.push(i);
            } else {
                randomIndexList.unshift(i);
            }
        }
    }
}
// 寫入排序順序
function assignIdx() {
    cardlist.value.forEach((ele, idx) => {
        ele.sortIdx = randomIndexList[idx];
    })
}
// 排序好要渲染的牌
const remderCards = computed(() => {
    return cardlist.value.sort((a, b) => {
        return a.sortIdx - b.sortIdx;
    })
})

onMounted(() => {
    randomIdx();
    assignIdx();
})
// 暫存前一個翻開的卡片訊息
const tempCard = ref({});
// 控制是否可以繼續翻牌
const canChoose = ref(true);
let count = ref(0);

function transCard(item) {
    // 如果還不能翻牌或是該點選的牌已經被翻開了
    if (canChoose.value === false || item.ispair === true) return;

    if (Object.keys(tempCard.value).length === 0) {
        // 紀錄要配對的第一張牌
        tempCard.value = item;
        item.ispair = true;
    } else {
        // 如果點擊速度太快點到同一張返回
        if (item.id === tempCard.value.id) return;
        // 禁止繼續翻牌
        canChoose.value = false;

        item.ispair = true;
        // 達到兩張都翻開後在判斷的效果
        setTimeout(() => {
            checkPair(item)
        }, 1000);
    }
}

function checkPair(item) {
    // 判斷是否配對成功
    if (item.cardText === tempCard.value.cardText) {

        // 配對成功 
        setTimeout(() => {
            // 清除暫存的第一張牌
            tempCard.value = {};
            // 允許繼續翻牌
            canChoose.value = true;
            count.value++;
        }, 1000);

    } else {
        // 配對失敗 >> 兩張卡牌恢復覆蓋狀態
        item.ispair = false;
        resetPrevCard();
    }
}
function resetPrevCard() {
    // 第一張牌恢復覆蓋狀態
    cardlist.value.forEach(ele => {
        if (ele.id === tempCard.value.id) {
            ele.ispair = false;

            // 清除第一張牌的暫存
            tempCard.value = {};

            // 允許繼續翻牌
            setTimeout(() => {
                canChoose.value = true;
            }, 1000);
        }
    })
}
//接收emitter方法
const emitter = inject("emitter")
const mission = true

// 監聽是否全部配對成功
// 傳送配對成功到todolist組件
watch(count, (nowValue) => {
    if (nowValue === 8) {
        console.log("恭喜挑戰遊戲成功!!!")
        emitter.emit('missionvalue', mission);
    }
})

// 重新一局遊戲
function restartGame() {
    console.log('重新開始')
    // 計數器歸零
    count.value = 0;

    // 清除第一張牌的暫存
    tempCard.value = {};

    // 重新寫入隨機順序
    randomIdx();
    assignIdx();

    // 還原蓋牌狀態
    cardlist.value.forEach(ele => {
        ele.ispair = false;
    })
}
</script>
<style>
        #pokerapp {
            width: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .pokerh1{
            text-align: center;
            font-weight: 100;
            font-size: 50px;
            color: rgb(29 123 152);
            margin-top: 100px;
            line-height: 1em;
        }
    
        .card {
            width: 120px;
            height: 160px;
            position: relative;
            cursor: pointer;
            perspective: 1000px;
            /*立體透視深度*/
            margin: 5px;
        }
    
        .back,
        .front {
            width: 100%;
            height: 100%;
            border-radius: 5px;
            border: 1px solid #777;
            position: absolute;
            backface-visibility: hidden;
            /*html 元素背後不可看見*/
            transition: .8s;
            /*動畫時間 0.8秒*/
        }
    
        .front {
            background-color: #fff;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 46px;
            font-weight: 700;
            font-family: "微軟正黑體";
            transform: rotateY(180deg);
        }
    
        .back {
            background-color: #eee;
            display: flex;
            justify-content: center;
            align-items: center;
            transform: rotateY(0deg);
        }
    
        .back::after {
            content: "";
            display: block;
            width: 87%;
            height: 92%;
            background: #fff;
            background-image: linear-gradient(45deg, red 26%, transparent 0, transparent 75%, red 0),
                linear-gradient(45deg, red 26%, transparent 0, transparent 75%, red 0);
            background-size: 10px 10px;
            background-position: 0 0, 15px 15px;
        }
    
        .transfront {
            transform: rotateY(0deg);
        }
    
        .transback {
            transform: rotateY(-180deg);
        }
    
        .cardtable {
            width: 520px;
            min-width: 520px;
            display: flex;
            flex-wrap: wrap;
            justify-content: space-around;
            position: relative;
        }
        .restart_wrap{
            width: 100%;
            height: 100%;
            background: rgba(245, 245 ,245, .9);
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 99;
            position: absolute;
            top: 0;
            left: 0;
            flex-wrap: wrap;
            align-content: center
        }
        .restart_wrap p{
            width: 100%;
            font-size: 2rem;
            display: block;
            text-align: center;
            color: rgb(161, 0, 0);
        }
        .replay {
            padding: 10px;
            font-size: 20px;
            font-weight: 700;
            border: 2px solid rgb(161, 0, 0);
            color: rgb(161, 0, 0);
            background: white;
            border-radius: 10px;
            cursor: pointer;
        }
</style>