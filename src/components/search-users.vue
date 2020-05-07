<template>
    <div class="users">
        <div class="users-counts">
            <p class="fnt">Red: {{allRedUsers}}</p>
            <p class="fnt">Yellow: {{allYellowUsers}}</p>
            <p class="fnt">Green: {{allGreenUsers}}</p>
        </div>

        <div class="users-search">
            <VueFuse
                    placeholder="Search users"
                    event-name="results"
                    :list="filteredUsers"
                    :keys="['user', 'access']"
                    class="users__field"
            />

            <div class="users-filters">
                <div v-for="(filter, index) in filters"
                     :key="index"
                     class="filter"
                     :class="[{'active': filter.active },`${filter.status}`]"
                     @click="filterUser(filter)"/>
            </div>
        </div>

        <div class="users-find fnt">
            Найдено
            <span v-if="result.length < users.length">
                {{ result.length }} из
            </span>
            {{users.length}}
        </div>

        <div class="users-list">
            <RecycleScroller
                    class="scroller"
                    :items="result"
                    :item-size="32"
                    :key-field="result.id"
                    v-slot="{ item }">
                <div class="user">
                    {{ item.user }}
                </div>
                <div class="users-filters">
                    <div class="filter"
                         :class="{'active': item.access === 'required' }"
                         @click="changeFilter(item.id, 'required')"/>
                    <div class="filter progress"
                         :class="{'active': item.access === 'progress' }"
                         @click="changeFilter(item.id, 'progress')"/>
                    <div class="filter accepted"
                         :class="{'active': item.access === 'accepted' }"
                         @click="changeFilter(item.id, 'accepted')"/>
                </div>
            </RecycleScroller>
        </div>
    </div>
</template>

<script>
    import Vue from 'vue'
    import VueFuse from 'vue-fuse'
    import {RecycleScroller} from 'vue-virtual-scroller'
    import randomUsers from '@/js/users'
    import {v4 as uuidv4} from 'uuid';

    Vue.use('RecycleScroller', RecycleScroller)
    Vue.use(VueFuse)

    //styles
    import 'vue-virtual-scroller/dist/vue-virtual-scroller.css'

    export default {
        name: "search-users",
        components: {
            RecycleScroller
        },
        data: () => ({
            users: [],
            result: [],
            userStatus: '',
            checkedFilter: '',
            filters: [
                {
                    status: 'required',
                    active: false
                },
                {
                    status: 'progress',
                    active: false
                },
                {
                    status: 'accepted',
                    active: false
                },
            ]
        }),
        created() {
            this.users = JSON.parse(localStorage.getItem('users'));

            if (!this.users) {
                this.setUsers();
                localStorage.setItem('users', JSON.stringify(this.users));
            }
            // call emit vue-fuse
            this.$on('results', results => {
                this.result = results
            })
        },
        computed: {
            filteredUsers() {
                let result = this.users

                if (this.userStatus) {
                    result = this.users.filter(el => el.access === this.userStatus);
                }
                return result;
            },
            allRedUsers() {
                return this.users.filter(el => el.access === 'required').length;
            },
            allYellowUsers() {
                return this.users.filter(el => el.access === 'progress').length;
            },
            allGreenUsers() {
                return this.users.filter(el => el.access === 'accepted').length;
            },
        },
        methods: {
            // initialize random list of users
            setUsers() {
                let firstNames = randomUsers.firstNames;
                let lastNames = randomUsers.lastNames;
                let accesses = randomUsers.accesses;
                let countUsers = 10000;

                let filledUsers = [];

                for (let i = 0; i < countUsers; i++) {
                    let user = {
                        id: uuidv4(),
                        user: firstNames[Math.floor(Math.random() * firstNames.length)] + ' ' +
                            lastNames[Math.floor(Math.random() * lastNames.length)],
                        access: accesses[Math.floor(Math.random() * accesses.length)]
                    }
                    filledUsers.push(user)
                }
                this.users = filledUsers;
            },
            filterUser(filter) {
                // set status
                this.userStatus = filter.status;

                this.filters.forEach(elem => {
                    elem.active = false;

                    if (elem.status === filter.status) {
                        elem.active = true;
                    }
                })
            },
            changeFilter(id, status) {
                // set status
                this.users.forEach(el => {
                    if (el.id === id) {
                        el.access = status;
                    }
                });

                localStorage.setItem('users', JSON.stringify(this.users));
            },

        }
    }
</script>

<style lang="css" scoped>
    /*reset*/
    * {
        box-sizing: border-box;
    }

    .scroller {
        height: 400px;
        overflow-y: scroll;
    }

    .user {
        height: 32%;
        padding: 0 12px;
        display: flex;
        align-items: center;
    }

    .users-counts {
        position: absolute;
        right: -100px;
        top: 100px;
    }

    .fnt {
        font-size: 14px;
        line-height: 17px;
        color: #2c3e50;
    }


    .users {
        position: relative;
        display: flex;
        flex-direction: column;
        align-items: center;
        padding: 20px;
        width: 100%;
        max-width: 444px;
        margin: 100px auto 0;
    }

    .users-search {
        position: relative;
        display: flex;
        margin-bottom: 10px;
        width: 100%;
    }

    .users__field {
        width: 100%;
        padding: 10px 145px 10px 20px;
        border: 1px solid black;
    }

    .users-filters {
        position: absolute;
        display: flex;
        align-items: center;
        justify-content: space-between;
        top: 50%;
        transform: translateY(-50%);
        right: 10px;
        width: 133px;
    }

    .filter {
        width: 37px;
        height: 24px;
        margin-left: 4px;
        background: red;
    }

    .users-filters .active {
        position: relative;
    }

    .users-filters .active:after {
        position: absolute;
        content: '';
        top: 0;
        left: 0;
        width: 37px;
        height: 24px;
        border: 4px solid blueviolet;
        box-sizing: border-box;
    }

    .users-filters .progress {
        background: greenyellow;
    }

    .users-filters .accepted {
        background: green;
    }

    .users-filters .filter:last-child {
        margin-left: 0;
    }

    .users-find {
        margin-bottom: 10px;
        width: 100%;
        text-align: right;
    }

    .users-list {
        padding: 10px;
        width: 100%;
        border: 1px solid black;
    }

    .user {
        display: flex;
        align-items: center;
        justify-content: space-between;
        width: 100%;
        margin-bottom: 10px;
    }

    .user-accesses {
        display: flex;
        align-items: center;
        justify-content: space-between;
        width: 80px;
    }

    .user:last-child {
        margin-bottom: 0;
    }


</style>