<template>
<div>
	<a @click='ExpandTask' :class='task_style'>
		{{ task.desc }}
	</a>

	<transition name='fade'>
		<div ref='expandedinfo' class='expanded-info' v-if='is_expanded'>
			<div>
				<a @click='ChangeTaskStatus(task)'>👌 Valmis</a> //
				<a @click='DeleteTask(task)'>💀 Poista</a>
			</div>

			<transition name='fade' mode='out-in' @enter='ItemAppears($event)'>
				<div v-if='!is_editing' class='description' @click='is_editing = true' key='notediting'>
					<a class='description'>
					<div class='notes' v-if='task.notes'>{{ task.notes }}</div>
					<div v-else><i>[Lisää kuvaus]</i></div>
					</a>
				</div>
				<div v-else key='editing' class='task-info'>
					<textarea class='textarea'
						v-model.trim='task.notes'
						@keyup.ctrl.enter='UpdateTask'
						@keyup.esc='is_editing = false'
						@blur='LoseFocus($event)'>
					</textarea>
					<!--<input type='button' value='Muokkaa' class='button' @click='UpdateTask'>-->
				</div>
			</transition>

			<ul>
				<ol v-for='(link, index) in task.links' :key='index'>
					<a class='link' :href='link[0]'>{{ link[1] }}</a>
				</ol>
			</ul>

			<ul v-if='task.session'>
				<a target='_blank' :href='get_session'>
					Avaa web-työtila ( {{ task.session.length }} tabia )
				</a>
			</ul>
		</div>

	</transition>
</div>
</template>

<script>
import server from '../api'

export default {
	name: 'TaskItem',
	data() {
		return {
			is_expanded: false,
			is_editing: false,
		}
	},
	props: ['task'],
	computed: {
		get_session() {
			const links = []
			for (const part of this.task.session) {
				links.push(part[0])
			}
			return 'ext+session://' + links.join(';;;')
		},
		task_style() {
			return this.task.status ? 'activetask' : 'inactivetask'
		}
	},
	filters: {
		format(text) {
			return text
		}
	},
	methods: {
		ItemAppears(element) {
			const textarea = element.getElementsByTagName('textarea')

			if (textarea.length) {
				element.getElementsByTagName('textarea')[0].focus()
			}
		},
		ChangeTaskStatus(task_data) {
			const new_data = task_data
			new_data.status = !new_data.status

			server.Put('task', new_data, () =>
				this.task.status = new_data.status)
		},
		DeleteTask(task) {
			if (confirm('Haluatko poistaa tehtävän?')) {
				server.Delete('task', task, () =>
					this.$emit('destroy'))
			}
		},
		ExpandTask() {
			this.is_expanded = !this.is_expanded
			this.is_editing = false
		},
		LoseFocus(element) {
			if (document.activeElement !== element.target) {
				this.UpdateTask()
				this.is_editing = false
			}
		},
		UpdateTask() {
			server.Put('task', this.task, () =>
				this.is_editing = false)
		},
	}
}
</script>

<style scoped>
.inactivetask {
	text-decoration: line-through;
	color: #707070;
}
div.description {
	margin: 10px 12px 18px;
}
.description {
	color: #D0D0D0;
	font-weight: none;
}
.notes {
	white-space: pre-wrap;
}
.fade-enter, .fade-leave-to {
	opacity: .3;
}
a.link {
	color: #30D030;
}
a.link:hover {
	color: #C0F090;
}
.textarea {
	height: 200px;
}
</style>
