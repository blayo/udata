<template>
<div>
    <div class="list-group-item" :id="discussionIdAttr" @click="toggleDiscussions"
        :class="{expanded: detailed}">
        <div class="format-label pull-left">
            <avatar :user="discussion.user"></avatar>
        </div>
        <span class="list-group-item-link">
            <a href="#{{ discussionIdAttr }}"><span class="fa fa-link"></span></a>
        </span>
        <h4 class="list-group-item-heading ellipsis open-discussion-thread">
            <span>{{ discussion.title }}</span>
            <span v-if="discussion.closed" class="fa fa-microphone-slash"></span>
        </h4>
        <p class="list-group-item-text ellipsis open-discussion-thread list-group-message-number-{{ discussion.id }}">
            <span v-if="!discussion.closed">{{ _('Discussion started on {created} with {count} messages.',
                 {created: createdDate, count: discussion.discussion.length})
            }}</span>
            <span v-if="discussion.closed">{{ _('Discussion closed on {closed} with {count} messages.',
                 {closed: closedDate, count: discussion.discussion.length})
            }}</span>
        </p>
    </div>
    <div v-for="(index, response) in discussion.discussion" id="{{ discussionIdAttr }}-{{ index }}"
        class="list-group-item list-group-indent animated discussion-messages-list"
        :class="{'body-only': index == 0}" v-show="detailed">
        <template v-if="index > 0">
            <div class="format-label pull-left">
                <avatar :user="response.posted_by"></avatar>
            </div>
            <span class="list-group-item-link">
                <a href="#{{ discussionIdAttr }}-{{ index }}"><span class="fa fa-link"></span></a>
            </span>
        </template>
        <p class="list-group-item-heading">
            {{{ response.content | markdown }}}
        </p>
    </div>
    <a v-if="!discussion.closed"
        class="list-group-item add new-comment list-group-indent animated"
        v-show="!formDisplayed && detailed" @click="displayForm">
        <div class="format-label pull-left">+</div>
        <h4 class="list-group-item-heading">
            {{ _('Add a comment') }}
        </h4>
    </a>
    <div v-el:form id="{{ discussionIdAttr }}-new-comment" v-show="formDisplayed" v-if="currentUser"
        class="list-group-item list-group-form list-group-indent animated">
        <div class="format-label pull-left">
            <avatar :user="currentUser"></avatar>
        </div>
        <span class="list-group-item-link">
            <a href="#{{ discussionIdAttr }}-new-comment"><span class="fa fa-link"></span></a>
            <a @click="hideForm"><span class="fa fa-times"></span></a>
        </span>
        <h4 class="list-group-item-heading">
            {{ _('Commenting on this thread') }}
        </h4>
        <p class="list-group-item-text">
            {{ _("You're about to answer to this particular thread about:") }}<br />
            {{ discussion.title }}
        </p>
        <thread-form v-ref:form :discussion-id="discussion.id"></thread-form>
    </div>
</div>
</template>

<script>
import config from 'config';
import Avatar from 'components/avatar.vue';
import ThreadForm from 'components/discussions/thread-form.vue';
import moment from 'moment';


export default {
    components: {Avatar, ThreadForm},
    props: {
        discussion: Object,
        position: Number,
    },
    data() {
        return {
            detailed: false,
            formDisplayed: false,
            currentUser: config.user,
        }
    },
    events: {
        'discussion:updated': function(discussion) {
            // Hide the form on comment submitted
            this.hideForm()
            return true; // Don't stop propagation
        }
    },
    computed: {
        discussionIdAttr() {
            return `discussion-${this.discussion.id}`;
        },
        createdDate() {
            return moment(this.discussion.created).format('LL')
        },
        closedDate() {
            return moment(this.discussion.closed).format('LL')
        }
    },
    methods: {
        toggleDiscussions() {
            this.detailed = !this.detailed;
        },
        /**
         * Display the comment form or triggers an authentication if required
         */
        displayForm() {
            this.$auth(this._('You need to be logged in to comment.'));
            this.formDisplayed = true; // Form is at the end of the expanded discussion
            this.detailed = true;
        },
        /**
         * Hide the comment form
         */
        hideForm() {
            this.formDisplayed = false;
        },

        /**
         * Trigger a new prefilled comment.
         */
        start(comment) {
            this.displayForm()
            // Wait for next tick because the form needs to be visible to scroll
            this.$nextTick(() => {
                if (this.$els.form && this.$refs.form) { // Avoid logging errors
                    this.$scrollTo(this.$els.form);
                    this.$refs.form.prefill(comment);
                }
            })
        },

        /**
         * Focus the thread or a specific comment if position is given
         */
        focus(index) {
            this.detailed = true;
            if (index) {
                this.$nextTick(() => {
                    this.$scrollTo(`#${this.discussionIdAttr}-${index}`);
                })
            } else {
                this.$scrollTo(this);
            }
        }
    }
}
</script>
