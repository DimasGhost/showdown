<template>
    <div class="content-panels-row">
        <card-link img="/game.png"
                   :title="match.game.name"
                   :to="`/games/${match.game.id}`"
                   class="card"
                   v-if="show_game"/>
        <div class="content-panel match-preview">
            <div class="participant participant-left">
                <router-link :to="`/bots/${left.id}`" class="bot jump-link">
                    <img class="bot-logo" src="/bot.png">
                    <div class="bot-name">{{left.name}}</div>
                </router-link>
                <router-link :to="`/users/${left.owner.id}`" class="user jump-link">
                    <img class="user-logo" src="/user.png">
                    <div class="user-login">{{left.owner.login}}</div>
                </router-link>
            </div>
            <div class="middle-info">
                <div class="match-result" :class="left_rerank_class" v-if="left_rerank !== undefined">
                    {{left_rerank}}
                </div>
                <div class="match-info">
                    <router-link :to="`/matches/${match.id}`" class="jump-link">
                        Перейти к матчу
                    </router-link>
                    <div class="match-state" :class="state_class">{{match_state}}</div>
                </div>
                <div class="match-result" :class="right_rerank_class" v-if="right_rerank !== undefined">
                    {{right_rerank}}
                </div>
            </div>
            <div class="participant participant-right">
                <router-link :to="`/bots/${right.id}`" class="bot jump-link">
                    <div class="bot-name">{{right.name}}</div>
                    <img class="bot-logo" src="/bot.png">
                </router-link>
                <router-link :to="`/users/${right.owner.id}`" class="user jump-link">
                    <div class="user-login">{{right.owner.login}}</div>
                    <img class="user-logo" src="/user.png">
                </router-link>
            </div>
        </div>
    </div>
</template>

<style lang="scss" scoped>
    .card {
        width: 123.5px;
        height: 123.5px;
        font-size: 14px;
    }

    .match-preview {
        flex-direction: row;
        justify-content: space-between;
        flex-grow: 1;
    }

    .middle-info {
        display: flex;
        align-items: center;
    }

    .match-result {
        font-size: 20px;
        &.positive {
            color: $col-good;
        }
        &.negative {
            color: $col-danger;
        }
    }

    .match-info {
        display: flex;
        flex-direction: column;
        align-items: center;
        margin-left: 50px;
        margin-right: 50px;
    }

    .match-state {
        margin-top: 20px;
        &.created {
            color: transparentize($col-active, 0.5);
        }
        &.inprocess {
            color: $col-active;
        }
        &.finished {
            color: $col-good;
        }
        &.cancelled {
            color: $col-danger;
        }
    }

    .participant {
        width: 30%;
        display: flex;
        flex-direction: column;
    }

    .participant-left {
        align-items: flex-start;
    }

    .participant-right {
        align-items: flex-end;
    }

    .bot {
        display: flex;
        align-items: center;
    }

    .bot-logo {
        width: 50px;
        height: 50px;
    }

    .bot-name {
        margin-left: 10px;
        margin-right: 10px;
    }

    .user {
        margin-top: 10px;
        display: flex;
        align-items: center;
    }

    .user-logo {
        width: 25px;
        height: 25px;
        margin-left: 12.5px;
        margin-right: 12.5px;
    }

    .user-login {
        font-size: 14px;
        margin-left: 10px;
        margin-right: 10px;
    }
</style>

<script>
    import CardLink from "@/components/CardLink.vue";

    export default {
        props: {
            match: Object,
            show_game: { default: false },
            prefer_left_bot_id: { default: null },
            prefer_left_user_id: { default: null }
        },
        methods: {
            format_rerank(id) {
                let res = this.rerank[id];
                if (res === undefined)
                    return undefined;
                res = res.toFixed(0);
                if (res >= 0)
                    res = `+${res}`;
                return res;
            },
            get_rerank_class(id) {
                let res = this.rerank[id];
                if (res === undefined)
                    return "";
                if (res > 0)
                    return "positive";
                if (res < 0)
                    return "negative";
            }
        },
        computed: {
            participants() {
                let first = this.match.participants[0], second = this.match.participants[1];

                if (this.prefer_left_bot_id === null && this.prefer_left_user_id === null)
                    return [first, second];

                if (this.prefer_left_bot_id !== null && this.prefer_left_bot_id.toString() === first.id.toString())
                    return [first, second];

                if (this.prefer_left_user_id !== null && this.prefer_left_user_id.toString() === first.owner_id.toString())
                    return [first, second];

                return [second, first];
            },
            left() {
                return this.participants[0];
            },
            right() {
                return this.participants[1];
            },
            rerank() {
                return this.match.results.rerank || {}
            },
            left_rerank() {
                return this.format_rerank(this.left.id);
            },
            left_rerank_class() {
                return this.get_rerank_class(this.left.id);
            },
            right_rerank() {
                return this.format_rerank(this.right.id);
            },
            right_rerank_class() {
                return this.get_rerank_class(this.right.id);
            },
            match_state() {
                let states = [
                    "-",
                    "СОЗДАН",
                    "В ПРОЦЕССЕ",
                    "ЗАВЕРШЕН",
                    "ОТМЕНЕН"
                ];
                let res = states[this.match.state];
                let game_results = this.match.results.game_results;
                if (game_results !== undefined) {
                    let ls = game_results[this.left.id];
                    let rs = game_results[this.right.id];
                    res += ` (${ls} : ${rs})`;
                }
                return res;
            },
            state_class() {
                let states = [
                    "",
                    "created",
                    "inprocess",
                    "finished",
                    "cancelled"
                ];
                return states[this.match.state];
            }
        },
        components: {
            CardLink
        }
    }
</script>