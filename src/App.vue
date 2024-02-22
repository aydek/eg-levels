<template>
    <img v-if="!altInWindow()" class="absolute w-screen h-screen top-0 left-0" src="/assets/devbg.png" />
    <div class="absolute bg-[#0E0E0EB2] w-full h-screen text-white flex items-center">
        <!-- All background images -->
        <img class="absolute bottom-0 left-0 h-full" src="/assets/bottomElipse.svg" />
        <img class="absolute top-0 right-0 h-full" src="/assets/topElipse.svg" />
        <img class="absolute top-0 right-0 h-full" src="/assets/gta_dude.png" />

        <!-- Main container -->
        <div class="ml-32 z-10 w-4/6">
            <div class="flex justify-between items-center pr-20">
                <div>
                    <div class="font-semibold text-4xl">{{ name.title }}</div>
                    <div class="text-gray-500">{{ name.subtext }}</div>
                </div>
                <div class="w-1/4 flex flex-col items-end text-sm">
                    <div class="text-base">
                        {{ PlayerActivityLevel }} lygis <span class="text-[#FFB800]">({{ PlayerActivityXP }} XP)</span>
                    </div>
                    <div>
                        <span>{{ getCurrentReq().from }} / {{ getCurrentReq().to }}</span>
                        <span class="text-[#FFB800] font-semibold"> XP</span>
                    </div>
                    <div class="relative w-full h-2 mt-1">
                        <div class="absolute h-full w-full left-0 top-0 bg-[#B3B3B3] rounded-lg"></div>
                        <div
                            :style="{
                                width: `${getBarWidth()}%`,
                            }"
                            class="absolute h-full left-0 top-0 bg-[#FFB800] rounded-lg"
                        ></div>
                    </div>
                </div>
            </div>
            <div class="grid grid-cols-5 overflow-y-scroll mt-5 h-[42rem] gap-7 pr-10">
                <div
                    v-for="(level, index) in levels"
                    :key="index + name.title"
                    class="bg-[#6d6d6d6c] w-full aspect-[4/5] border broder-[#B3B3B3] rounded-lg relative p-2 flex flex-col items-center justify-between text-sm"
                >
                    <!-- If level has not been reach tint item -->
                    <div
                        v-if="PlayerActivityLevel < level.Level"
                        class="bg-[#00000080] w-full h-full left-0 top-0 rounded-lg z-10 absolute"
                    ></div>

                    <!-- Level has been reached and prize not taken yet -->

                    <div
                        class="bg-[#3E3E3EAA] w-full h-full left-0 top-0 rounded-lg z-10 absolute grid place-items-center"
                        v-if="
                            level.Level <= PlayerActivityLevel &&
                            level.Reward &&
                            TakenRewards &&
                            !TakenRewards.includes(level.Level)
                        "
                        @click="takeReward(level.Level)"
                    >
                        <button class="border bg-[#FFB8001A] border-[#FFB800] rounded-3xl p-3 w-3/4 animate-pulse">
                            Atsiimti prizą
                        </button>
                    </div>

                    <!-- Level item ui -->

                    <div>{{ level.Level }} LYGIS ({{ level.NeedXp }} XP)</div>

                    <img class="w-24 aspect-square" src="/assets/worm.png" />

                    <!-- Used to auto scrool to current level -->
                    <div v-if="level.Level === PlayerActivityLevel" ref="autoScroll"></div>

                    <div class="text-[#FFB800]">{{ level.LevelPerks[2] }}</div>

                    <img src="/assets/divider.svg" />
                    <ul class="text-center text-xs">
                        <p class="text-[#FFB800] mr-1">BONUS</p>
                        <li
                            class="flex items-center justify-center"
                            v-for="(perk, index) in level.LevelPerks"
                            :key="index + perk"
                        >
                            <div v-if="index !== 2">
                                <span class="text-[#FFB800] mr-1">•</span>
                                {{ perk }}
                                <span class="text-[#FFB800] ml-1">•</span>
                            </div>
                        </li>
                    </ul>
                </div>
            </div>

            <ul class="text-sm mt-4">
                <li v-for="(item, index) in details" :key="index + item.text">
                    <span>{{ item.text }}</span>
                    <span class="text-[#FFB800] ml-1"> {{ item.value }}</span>
                </li>
            </ul>
        </div>
    </div>
</template>
<script>
export default {
    name: 'Varzybos',
    data() {
        return {
            name: { title: 'UOGAVIMO LYGIAI', subtext: 'Kelkis lygį ir gauk prizus!' },
            details: [
                {
                    text: '• Radus mėlynių, spanguolių arba žemuogių',
                    value: '+1 XP',
                },
                {
                    text: '• Radus aviečių arba gervuogių',
                    value: '+2 XP',
                },
                {
                    text: '• Esant aktyviai veiklai',
                    value: 'X2 XP',
                },
            ],

            levels: [],
            PlayerActivityLevel: 99,
            PlayerActivityXP: 9920,
            TakenRewards: [],
        };
    },
    methods: {
        updateData(jsonData, jsonData2) {
            let parsed = JSON.parse(jsonData);
            this.PlayerActivityLevel = parsed.PlayerActivityLevel;
            this.PlayerActivityXP = parsed.PlayerActivityXP;
            this.TakenRewards = parsed.TakenRewards;

            this.levels = JSON.parse(jsonData2);
        },
        takeReward(levelIndex) {
            this.TakenRewards.push(levelIndex);
            if ('alt' in window) {
                alt.emit('TakeLevelReward', 2, levelIndex);
            }
        },
        getBarWidth() {
            return (100 / this.getCurrentReq().to) * this.getCurrentReq().from;
        },
        getCurrentReq() {
            const current = this.levels.find((item) => item.Level === this.PlayerActivityLevel);
            const next = this.levels.find((item) => item.Level === this.PlayerActivityLevel + 1);
            let from = 0;
            let to = 0;
            if (current && next) {
                to = next.NeedXp - current.NeedXp;
                from = to - (next.NeedXp - this.PlayerActivityXP);
            }
            return { from, to };
        },
        altInWindow() {
            return 'alt' in window;
        },
    },
    watch: {
        levels(newValue, oldValue) {
            this.$nextTick(() => {
                const element = this.$refs['autoScroll'];

                if (element) {
                    element[0].scrollIntoView({ block: 'center' });
                }
            }, 100);
        },
    },
    mounted() {
        if ('alt' in window) {
            this.updateData(this.$route.params.jsonData, this.$route.params.json2Data);
        } else {
            const generatedLevels = [];
            for (let level = 100; level >= 1; level--) {
                const needXp = 100 * level;
                const levelData = {
                    Level: level,
                    NeedXp: needXp,
                    LevelPerks: ['+0.5 Greičiui', '+0.5 Sėkmei', 'Papildomas Item'],
                    Reward: true,
                };
                generatedLevels.push(levelData);
            }
            this.levels = generatedLevels;
        }
    },
};
</script>
<style scoped>
::-webkit-scrollbar-track {
    background-color: transparent;
}

::-webkit-scrollbar {
    width: 4px;
    border-radius: 20px;
    background-color: #737373;
    border: 1px solid transparent;
    background-clip: content-box;
}

::-webkit-scrollbar-thumb {
    background-color: #ffb800;
    border-radius: 20px;
}
</style>
