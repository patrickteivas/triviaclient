<template>
    <div >


    <div class="container" v-if="!started">
        <div class="columns">
            <div class="column">
                Join Code is {{code}}
            </div>
            <div class="column is-one-fifth">
                <player-list :players="players"></player-list>
            </div>
        </div>
        <button @click="start">Start</button>

    </div>
        <div class="container" v-if="question">
            <h1 class="is-size-1" v-html="question"></h1>
            <h1 v-if="timer">{{timer}}</h1>
        </div>
        <div class="container" v-if="sortedScore.length">
            <ul>
                <li v-for="points in sortedScore">{{points.name}} - {{points.score}}</li>
            </ul>
        </div>
    </div>
</template>
<script>
    import PlayerList from "./components/PlayerList";
    export default {
        components: {PlayerList},
        created() {
            this.ws = new WebSocket('ws://localhost:8080');
            this.ws.onopen = (event) => {
                this.ws.send(JSON.stringify({sender:'game', action:'getCode'}));
            };
            this.ws.onmessage = (event) => {
                let msg = JSON.parse(event.data);
                console.log(msg);
                if(msg.action === 'code'){
                    this.code = msg.data.code;
                }
                if(msg.action === 'playerJoined'){
                    this.players.push(msg.data.name);
                }
                if(msg.action === 'uid'){
                    this.uid = msg.data.uid;
                }
                if(msg.action === 'question'){
                    this.question = msg.data.question;
                    this.timer = 30;
                }
                if(msg.action === 'timeout'){
                    this.timer--;
                }
                if(msg.action === 'score'){
                    this.question = '';
                    this.score = msg.data.score;
                }
            };
        },
        data(){
            return {
                players: [],
                code: '',
                uid: '',
                ws: null,
                started: false,
                question: null,
                timer: 0,
                score: []
            }
        },
        computed: {
            sortedScore() {
                return this.score.sort((a, b) => parseFloat(b.score) - parseFloat(a.score));
            }
        },
        methods: {
            start(){
                this.ws.send(JSON.stringify({sender: 'game', action:'start', id: this.uid}));
                this.started = true;
            }
        }
    }
</script>