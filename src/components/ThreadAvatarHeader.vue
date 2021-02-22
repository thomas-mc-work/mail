<template>
	<div ref="avatarHeader" class="avatar-header">
		<div v-if="label" class="label">
			<span>{{ label }}</span>
		</div>
		<!-- Participants that can fit in the parent div -->
		<RecipientBubble v-for="participant in participants.slice(0,participantsToDisplay)"
			:key="participant.email"
			:email="participant.email"
			:label="participant.label" />
		<!-- Indicator to show that there are more participants than displayed -->
		<Popover v-if="participants.length > participantsToDisplay" class="avatar-more">
			<span slot="trigger">
				{{ moreParticipantsString }}
			</span>
			<RecipientBubble v-for="participant in participants.slice(participantsToDisplay)"
				:key="participant.email"
				:email="participant.email"
				:label="participant.label" />
		</Popover>
		<!-- Remaining participants, if any (Needed to have avatarHeader reactive) -->
		<RecipientBubble v-for="participant in participants.slice(participantsToDisplay)"
			:key="participant.email"
			class="avatar-hidden"
			:email="participant.email"
			:label="participant.label" />
	</div>
</template>

<script>
import Popover from '@nextcloud/vue/dist/Components/Popover'
import RecipientBubble from './RecipientBubble'
import debounce from 'lodash/fp/debounce'

export default {
	name: 'ThreadAvatarHeader',
	components: {
		RecipientBubble,
		Popover,
	},
	props: {
		label: {
			type: String,
			required: false,
			default: '',
		},
		participants: {
			type: Array,
			required: true,
		},
	},
	data() {
		return {
			participantsToDisplay: 999,
			resizeDebounced: debounce(500, this.updateParticipantsToDisplay),
		}
	},
	computed: {
		moreParticipantsString() {
			// Returns a number showing the number of thread participants that are not shown in the avatar-header
			return `+${this.participants.length - this.participantsToDisplay}`
		},
	},
	watch: {
		participants() {
			this.updateParticipantsToDisplay()
		},
	},
	created() {
		window.addEventListener('resize', this.resizeDebounced)
	},
	mounted() {
		this.updateParticipantsToDisplay()
	},
	beforeDestroy() {
		window.removeEventListener('resize', this.resizeDebounced)
	},
	methods: {
		updateParticipantsToDisplay() {
			// Wait until everything is in place
			if (!this.$refs.avatarHeader || !this.participants) {
				return
			}

			// Only include dom element nodes and ignore avatar-more span
			const children = Array
				.from(this.$refs.avatarHeader.childNodes)
				.filter((node) => node.nodeType === Node.ELEMENT_NODE)
				.filter((node) => !node.classList.contains('avatar-more'))

			// Reserve 100px for the avatar-more span
			const maxWidth = this.$refs.avatarHeader.clientWidth - 100

			let childrenWidth = 0
			let fits = 0

			// Calculate full width including margins
			const getFullWidth = (node) => {
				const styles = window.getComputedStyle(node)
				const margins = parseFloat(styles.marginLeft) + parseFloat(styles.marginRight)
				return node.clientWidth + margins
			}

			// Measure label width if present
			if (this.label) {
				childrenWidth += getFullWidth(children[0])
				children.splice(0, 1)
			}

			for (const child of children) {
				const newWidth = childrenWidth + getFullWidth(child)
				if (newWidth > maxWidth) {
					break
				}

				fits++
				childrenWidth = newWidth
			}

			this.participantsToDisplay = fits
		},
	},
}
</script>

<style lang="scss" scoped>
.avatar-header {
	max-height: 24px;
	overflow: hidden;
}

.avatar-more {
	display: inline-block;
	vertical-align: middle;

	span {
		display: inline-block;
		vertical-align: top;
		height: 20px;
		line-height: 20px;
		background-color: var(--color-background-dark);
		border-radius: 10px;
		cursor: pointer;
		padding: 0 4px;
	}
}

.avatar-hidden {
	visibility: hidden;
}

.popover__wrapper {
	max-width: 500px;
}

::v-deep .user-bubble__wrapper:not(:last-child) {
	margin-right: 4px;
}

::v-deep .user-bubble__title {
	cursor: pointer;
}

.label {
	display: inline-block;
	margin-right: 2px;
	vertical-align: middle;

	span {
		height: 20px;
		line-height: 20px;
		vertical-align: top;
	}
}
</style>
