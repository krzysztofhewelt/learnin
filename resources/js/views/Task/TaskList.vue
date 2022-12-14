<template>
    <div class="flex justify-between">
        <div class="flex flex-col space-y-3 lg:flex-row lg:space-x-10 lg:space-y-0">
            <Tab :active="currentTab === 'active'"
                 @click="setTasksView('active')">
                {{ $t('task.active_tab') }} ({{ getActiveTasks.length }})
            </Tab>
            <Tab :active="currentTab === 'ended'"
                 @click="setTasksView('ended')">
                {{ $t('task.ended_tab') }} ({{ getExpiredTasks.length }})
            </Tab>
            <Tab :active="currentTab === 'marked'"
                 v-if="isStudent"
                 @click="setTasksView('marked')">
                {{ $t('task.marked_tab') }} ({{ getMarkedTasks.length }})
            </Tab>
            <Tab :active="currentTab === 'upcoming'"
                 v-if="isTeacher"
                 @click="setTasksView('upcoming')">
                {{ $t('task.upcoming_tab') }} ({{ getUpcomingTasks.length }})
            </Tab>
        </div>
        <div>
            <router-link :to="{name:'TasksCreate'}"
                         v-if="isTeacher"
                         class="normal_btn">
                <Add class="w-4 h-4 m-1" />
                {{ $t('task.add_task') }}
            </router-link>
        </div>
    </div>

    <div v-if="category" class="py-2 text-xl font-bold">
        {{ $t('task.showing_tasks_for_category', {category: category}) }}
    </div>

    <div v-if="loading" class="mt-6 text-2xl">
        <Loading class="w-4 h-4 inline animate-spin" />
        {{ $t('general.loading') }}
    </div>
    <div v-if="result.length === 0 && !loading" class="mt-6 text-2xl font-bold">
        {{ $t('general.no_data') }}
    </div>

    <div class="mt-6" v-for="task in result" :key="task.id">
        <div class="relative grid grid-flow-row grid-cols-1 space-y-3 lg:space-y-0 w-full break-words rounded-lg bg-white shadow-xl p-6 lg:divide-x text-center" :class="[ (currentTab === 'marked') ? 'lg:grid-cols-4' : 'lg:grid-cols-3' ]">
            <div>
                <router-link :to="{name:'TasksView', params: {id:task.id}}" class="block font-bold text-purple-800 text-xl m-auto no-underline hover:text-purple-500">
                    {{ task.name }}
                </router-link>
                <div class="text-gray-500">
                    <b>{{ $t('course.course') }}:</b>
                    {{ task.course.name }}
                </div>
                <div class="text-gray-500">
                    <b>{{ $t('course.category') }}:</b>
                    {{ task.category.name }}
                </div>
            </div>
            <div>
                <div class="block text-xl font-bold">{{ $t('task.available_from_label') }}</div>
                <div class="text-2xl">{{ getFormattedDate(task.available_from) }}</div>
            </div>
            <div>
                <div class="block text-xl font-bold">{{ $t('task.available_to_label') }}</div>
                <div class="text-2xl">{{ getFormattedDate(task.available_to) || $t('general.not_available') }}</div>
            </div>
            <div v-if="currentTab === 'marked'">
                <div class="block text-xl font-bold">{{ $t('task.marked_task') }}</div>
                <div class="block text-zinc-400"><b>{{ $t('mark.mark') }}:</b> {{ task.user_marks[0].mark }}</div>
                <div class="block text-zinc-400"><b>{{ $t('mark.points') }}:</b> {{ task.user_marks[0].points }} / {{ task.max_points }}</div>
            </div>
        </div>
    </div>
</template>

<script>
import {mapActions, mapGetters, mapState} from "vuex"
import Loading from "../../components/Icons/Loading"
import Tab from "../../components/Tab";
import Add from "../../components/Icons/Add";

export default {
    name: "TaskList",
    components: {Add, Tab, Loading},
    data() {
        return {
            currentTab: 'active',
            category: '',
            result: []
        }
    },

    computed: {
        ...mapState('user', ['loading', 'tasks']),
        ...mapGetters('user', ['getActiveTasks', 'getExpiredTasks', 'getMarkedTasks', 'getUpcomingTasks', 'getFormattedDate']),
        ...mapGetters('login', ['isTeacher', 'isStudent'])
    },

    methods: {
        ...mapActions('user', ['getUserTasks', 'getCategoryTasks']),

        setTasksView(type) {
            switch(type) {
                case 'active': this.result = this.getActiveTasks; this.currentTab = 'active'; break
                case 'ended': this.result = this.getExpiredTasks; this.currentTab = 'ended'; break
                case 'marked': this.result = this.getMarkedTasks; this.currentTab = 'marked'; break
                case 'upcoming': this.result = this.getUpcomingTasks; this.currentTab = 'upcoming'; break
                default: this.result = this.getActiveTasks
            }
        }
    },

    created() {
        if(this.$route.name === 'TasksInCategory')
        {
            this.getCategoryTasks(this.$route.params.id)
                .then(
                    () => {
                        this.result = this.getActiveTasks
                    }
                )

            axios.get('/categories/show/' + this.$route.params.id)
                .then(
                    response => {
                        this.category = response.data.name
                    }
                )
        }
        else {
            this.category = ''

            this.getUserTasks()
                .then(
                    () => {
                        this.result = this.getActiveTasks
                    }
                )
        }
    }
}
</script>
