<script setup>
import { ref, computed } from 'vue';

const presets = [
    {
        name: "レインボー",
        steps: [
            {"color":"#ff0000","position":0},
            {"color":"#ffa500","position":17},
            {"color":"#ffff00","position":33},
            {"color":"#008000","position":50},
            {"color":"#00ffff","position":67},
            {"color":"#0000ff","position":83},
            {"color":"#800080","position":100}
        ]
    }
];

const colorSteps = ref([
    {
        color: "#000000",
        position: 0
    },
    {
        color: "#ffffff",
        position: 100
    }
]);

const presetIndex = ref("");

const text = ref("");

const addColorStep = () => {
    colorSteps.value.push({
        color: "",
        position: 0
    });
};

const equializeColorStep = () => {
    colorSteps.value.forEach((e, i) => {
        e.position = Math.round(100 / (colorSteps.value.length - 1) * i);
    });
};

const deleteColorStep = (index) => {
    colorSteps.value.splice(index, 1);
};

const applyPreset = (pI) => {
    if (isNaN(pI)) {
        return;
    }
    colorSteps.value = presets[pI].steps;
}

const CSSGradient = computed(() => {
    const cs = colorSteps.value;
    const gradientValues = cs.sort((a, b) => a.position - b.position).map((e) => e.color + " " + e.position.toString() + "%").join(",");
    return 'background-image: linear-gradient(to right, ' + gradientValues + ')';
});

const generatedText = computed(() => {
    if (text.value == "" || colorSteps.value.length < 2) return;
    const splitCount = colorSteps.value.map((e) => Math.round(e.position / 100 * text.value.length));
    let chunks = [];
    for (let index = 0; index < splitCount.length - 1; index++) {
        chunks[index] = text.value.substring(splitCount[index], splitCount[index + 1]);
    }
    return chunks.map((e, i) => {
        const beforeRGB = colorSteps.value[i]?.color.substring(1).match(/.{2}/g).map((f) => parseInt(f, 16));
        const afterRGB = colorSteps.value[i + 1]?.color.substring(1).match(/.{2}/g).map((f) => parseInt(f, 16)) || null;
        return e.split("").map((f, j, a) => {
            if (afterRGB == null) {
                return `$[fg.color=${colorSteps.value[i].color} ${f}]`;
            }
            const steppedRGB = beforeRGB.map((e, i) => (e - Math.floor((e - afterRGB[i]) / a.length * j)).toString(16).padStart(2, "0")).join("");
            return `$[fg.color=${steppedRGB} ${f}]`;
        }).join("");
    }).join("");
});

const copyText = () => {
    if (!navigator.clipboard) {
        alert("このブラウザは対応していません");
        return;
    }

    navigator.clipboard.writeText(generatedText.value).then(() => {
        alert("コピー成功");
    }).catch(() => {
        alert("コピーに失敗しました");
    })
}
</script>
<template>
    <div class="mt-3 container mb-5" id="app">
        <h2 class="mb-4">Misskey グラデーション文字作成ツール</h2>
        <div class="my-2 row">
            <label for="startColor" class="col-lg-2 col-sm-4 col-form-label">グラデーション作成</label>
            <div class="col-lg-10 col-sm-8">
                <div class="mb-2">
                    <div class="input-group">
                        <select class="form-select" v-model="presetIndex">
                            <option value="">【プリセットから選ぶ】</option>
                            <option v-for="item, index in presets" :value="index">{{item.name}}</option>
                        </select>
                        <button class="btn btn-outline-primary" @click="applyPreset(presetIndex)">適用</button>
                    </div>
                </div>
                <div class="d-flex mb-2" v-for="item, index in colorSteps">
                    <div class="flex-shrink-0 me-2">
                        <div class="input-group">
                            <input class="form-control form-control-color" style="width: 3rem;" type="color" v-model="item.color" />
                            <span class="input-group-text font-monospace">{{ item.color }}</span>
                        </div>
                    </div>
                    <div class="flex-grow-1 me-2">
                        <div class="input-group">
                            <input class="form-control" type="number" min="0" max="100" v-model="item.position" />
                            <span class="input-group-text">%</span>
                        </div>
                    </div>
                    <div class="flex-shrink-0">
                        <button class="btn btn-outline-danger" @click="deleteColorStep(index)" :disabled="colorSteps.length <= 2">削除</button>
                    </div>
                </div>
                <div class="mb-3">
                    <button class="btn btn-secondary me-1" @click="addColorStep()">グラデーション位置を追加</button>
                    <button class="btn btn-secondary mr-1" @click="equializeColorStep()">均等配置</button>
                </div>
                <div class="rounded border py-4" :style="CSSGradient">

                </div>
            </div>
        </div>
        <div class="mt-2 mb-4 row">
            <label for="startColor" class="col-lg-2 col-sm-4 col-form-label">ノートしたい文字</label>
            <div class="col-lg-10 col-sm-8">
                <input type="text" class="form-control" v-model="text" placeholder="いまどうしてる？" />
            </div>
        </div>
        <div class="my-2 row">
            <div class="col-lg-2 col-sm-4 col-form-label">
                出来上がり
            </div>
            <div class="col-lg-10 col-sm-8">
                <div class="input-group">
                    <input type="text" class="form-control" readonly disabled :value="generatedText" />
                    <button class="btn btn-outline-primary" @click="copyText()">コピー</button>
                    <a :href="`https://misskey.io/share?text=${encodeURIComponent(generatedText)}`" class="btn btn-success" target="_blank">ノートする</a>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
    transition: filter 300ms;
}

.logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
    filter: drop-shadow(0 0 2em #42b883aa);
}</style>
