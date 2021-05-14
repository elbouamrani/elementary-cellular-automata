<template>
    <div id="app">
        <div class="action-container">
            <label>
                <button @click="reset()">reset</button>
            </label>
            &nbsp;|&nbsp;
            <label>
                <button @click="nextTick()">next</button>
            </label>
            &nbsp;|&nbsp;
            <label>
                <span>seed </span>
                <select @change="selectSeed" v-model="seed">
                    <option v-for="(seed, key) in seedSet" :key="key">
                        {{ seed }}
                    </option>
                </select>
            </label>
            &nbsp;|&nbsp;
            <label>
                <span>running </span>
                <input type="checkbox" v-model="running" />
            </label>
            &nbsp;|&nbsp;
            <label>
                <span>rule </span>
                <input
                    type="number"
                    step="0"
                    v-model="ruleIndex"
                    style="width: 30pt"
                />
            </label>
            <hr />
            <label>
                <span>size </span>
                <input
                    type="number"
                    step="0"
                    v-model="size"
                    style="width: 30pt"
                />
            </label>
            &nbsp;|&nbsp;
            <label>
                <span>pixelSize </span>
                <input
                    type="number"
                    step="0"
                    v-model="pixelSize"
                    style="width: 30pt"
                    @change="setPixelSize"
                />
            </label>
            &nbsp;|&nbsp;
            <label>
                <span>screenSize </span>
                <input
                    type="number"
                    step="0"
                    :placeholder="screenSize"
                    @change="setScreenSize"
                    style="width: 30pt"
                />
            </label>
            &nbsp;|&nbsp;
            <label>
                <span>cycleDelay </span>
                <input
                    type="number"
                    step="0"
                    v-model="cycleDelay"
                    style="width: 40pt"
                    @change="setCycleDelay"
                />
            </label>
            <hr />
            <span
                v-for="(rule, key) in specialRules"
                :key="key"
                @click="setRuleIndex(rule)"
                style="padding: 2pt; cursor: pointer"
            >
                R{{ rule }}
            </span>
        </div>
        <hr />
        <div ref="screenContainer" class="screen-container">
            <Screen :world="world" />
        </div>
    </div>
</template>

<script>
import Screen from "./components/Screen";

export default {
    name: "App",
    components: {
        Screen,
    },
    data() {
        return {
            running: false,
            ticker: null,

            ruleIndex: 30,
            ruleSet: null,

            seed: "random",
            seedSet: ["random", "single", "mod 2", "mod 3", "mod 3 bar"],

            world: [],
            size: 100,
            cycleDelay: 300,
            cycle: 0,

            pixelSize: 3,
            screenSize: 50,

            specialRules: [
                7,
                9,
                18,
                22,
                26,
                30,
                37,
                57,
                60,
                73,
                90,
                106,
                110,
                150,
                184,
            ],
        };
    },
    methods: {
        getRule(array, index) {
            const left = this.getCell(array, index - 1);
            const center = this.getCell(array, index);
            const right = this.getCell(array, index + 1);

            const key = `R${left}${center}${right}`;

            // console.log(`${key} : ${this.ruleSet[key]}`);

            return this.ruleSet[key];
        },
        calculateRule() {
            const rules = [];
            const binaryArray = this.toBinaryString(this.ruleIndex, 8)
                .split("")
                .reverse();

            // console.log(`binary : ${binaryArray}`);

            for (let index = binaryArray.length - 1; index >= 0; index--) {
                const key = "R" + this.toBinaryString(index, 3);
                rules[key] = binaryArray[index];

                // console.log(`${index} : ${key} : ${binaryArray[index]}`);
            }

            this.ruleSet = rules;

            // console.log(rules);
            // console.log(this.ruleSet);
        },
        toBinaryString(number, padding) {
            return (number >>> 0).toString(2).padStart(padding, "0");
        },
        getCell(array, index) {
            return array[index] === undefined ? 0 : array[index];
        },
        update() {
            // console.log(this.ruleSet);

            const computeArray = [];
            const currentArray = this.world[this.world.length - 1];

            for (let index = 0; index < currentArray.length; index++) {
                computeArray[index] = this.getRule(currentArray, index);
            }

            this.world.push(computeArray);

            if (this.world.length > this.screenSize) {
                // this.world.shift();
                this.world.splice(0, this.world.length - this.screenSize);
            }

            this.cycle++;
        },
        reset() {
            this.world = [];
            this.init();
        },
        init() {
            this.seedWorld();
            this.calculateRule();
            this.setupCycle();
        },
        seedWorld() {
            const seed = [];
            for (let index = 0; index < this.size; index++) {
                // seed.push(Math.round(Math.random()));
                if (this.seed == "random") {
                    seed.push(Math.round(Math.random()));
                } else if (this.seed == "single") {
                    seed.push(index === Math.round(this.size / 2) ? 1 : 0);
                } else if (this.seed == "mod 2") {
                    seed.push(index % 2 ? 1 : 0);
                } else if (this.seed == "mod 3") {
                    seed.push(index % 3 ? 1 : 0);
                } else if (this.seed == "mod 3 bar") {
                    seed.push(index % 3 ? 0 : 1);
                }
            }
            this.world.push(seed);
        },
        setupCycle() {
            clearInterval(this.ticker);

            this.ticker = setInterval(() => {
                if (this.running) {
                    this.update();
                }
            }, this.cycleDelay);
        },
        nextTick() {
            this.update();
        },
        selectSeed(e) {
            this.seed = e.target.value;
        },
        setPixelSize() {
            // console.log(this.$refs.screenContainer);
            this.$refs.screenContainer.style.setProperty(
                "--pixel-size",
                this.pixelSize + "pt"
            );
        },
        setCycleDelay() {
            console.log(this.cycleDelay);
            this.setupCycle();
        },
        setScreenSize(event) {
            this.screenSize = event.target.value ?? 1;
            event.target.value = "";
        },
        setRuleIndex(ruleIndex) {
            this.ruleIndex = ruleIndex;
            this.reset();
            this.seed = "random";
            this.running = true;
        },
    },
    mounted() {
        this.init();
    },
};
</script>

<style>
.screen-container {
    --pixel-size: 3pt;
}
</style>