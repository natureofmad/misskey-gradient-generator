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
    },
    {
        name: "レインボー改",
        steps: [
            {"color":"#ff0000","position":0},
            {"color":"#ffa500","position":17},
            {"color":"#ffff00","position":33},
            {"color":"#00ff00","position":50},
            {"color":"#00ffff","position":67},
            {"color":"#0000ff","position":83},
            {"color":"#ff00ff","position":100}
        ]
    },
    {
        name: "黄桃",
        steps: [
            {color: "#fa709a", position: 0},
            {color: "#fee140", position: 100}
        ]
    },
    {
        name: "もえるオレンジ",
        steps: [
            {color: "#f83600", position: 0},
            {color: "#f9d423", position: 100}
        ]
    },
    {
        name: "サイバー青",
        steps: [
            {color: "#6a11cb", position: 0},
            {color: "#2575fc", position: 100}
        ]
    },
    {
        name: "サイバーピンク",
        steps: [
            {color: "#c471f5", position: 0},
            {color: "#fa71cd", position: 100}
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
    colorSteps.value.sort((a, b) => a.position - b.position).forEach((e, i) => {
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
    colorSteps.value = JSON.parse(JSON.stringify(presets[pI].steps));
}

function sanitizeHTMLString(str) {
    return str.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/"/g, '&quot;').replace(/'/g, '&#39;');
}

const CSSGradient = computed(() => {
    const cs = JSON.parse(JSON.stringify(colorSteps.value));
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

const generatedHTML = computed(() => {
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
                return `<span style="color: #${colorSteps.value[i].color}">${sanitizeHTMLString(f)}</span>`;
            }
            const steppedRGB = beforeRGB.map((e, i) => (e - Math.floor((e - afterRGB[i]) / a.length * j)).toString(16).padStart(2, "0")).join("");
            return `<span style="color: #${steppedRGB}">${sanitizeHTMLString(f)}</span>`;
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
                            <input class="form-control form-control-color flex-grow-0" style="width: 3rem;" type="color" v-model="item.color" />
                            <input class="form-control font-monospace" maxlength="7" type="text" style="width: calc(8ch + 1.5rem);" v-model="item.color" />
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
                <div id="emailHelp" class="form-text">ここでは平文のみを入力（その他の書式設定は生成後に行う）</div>
            </div>
        </div>
        <div class="my-2 row">
            <div class="col-lg-2 col-sm-4 col-form-label">
                プレビュー
            </div>
            <div class="col-lg-10 col-sm-8">
                <div class="rounded border px-2 text-preview" v-html="generatedHTML"></div>
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
        <div class="my-2 row align-items-center">
            <div class="col-lg-2 col-sm-4 col-form-label">
                ついでに作者をフォロー<br>
                （大感謝）
            </div>
            <div class="col-lg-10 col-sm-8 d-flex align-items-center position-relative">
                <img class="avatar rounded-circle me-2" src="/icons.jpg" />
                <div class="profile">
                    <div>とんこう科学部（大阪府立富田林中学校・高等学校科学部）</div>
                    <div>@natureofmad_@misskey.io</div>
                </div>
                <a class="btn btn-outline-success stretched-link" target="_blank" href="https://misskey.io/@natureofmad_">フォロー</a>
            </div>
        </div>
    </div>
</template>

<style scoped>
.text-preview {
    font-size: 1.5rem;
    line-height: 3rem;
    height: calc(3rem + 20px);
    padding-bottom: 20px;
    overflow-x: scroll;
    overflow-y: hidden;
    white-space: nowrap;
    font-weight: 700;
}
.avatar {
    height: 32px;
}
.profile div:nth-child(1) {
    font-weight: 700;
    font-size: 1.1em;
    line-height: 1.2em;
}
.profile div:nth-child(2) {
    font-size: 0.9em;
    color: #636363;
}
</style>
