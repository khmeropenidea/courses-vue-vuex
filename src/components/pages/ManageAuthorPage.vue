<template>
    <div>
        <AuthorForm
            :author="author"
            :errors="errors"
            :onSave="handleSaveAuthor"
            :onChange="handleUpdateAuthorState"
            :onDelete="handleDeleteAuthor"
            :saving="saving"
            :deleting="deleting"
        />
    </div>
</template>


<script>
    import AuthorForm from '../AuthorForm';
    import toastr from 'toastr';
    import { mapState } from 'vuex';
    import { fetchAuthor, resetErrors } from '../../store/actionCreators';

    export default {
        name: 'ManageAuthorPage',

        props: [
            'id'
        ],

        components: {
            AuthorForm
        },

        beforeMount () {
            // clear errors
            this.$store.dispatch(resetErrors());

            if (this.$route.params.id && this.$route.params.id.length > 0) {
                this.$store.dispatch(fetchAuthor(this.$route.params.id))
                    .then(() => {
                        // load author record if we don't already have it
                        // already handled in fetchAuthor (this.author is the copy in the store)
                        //this.$store.commit('LOAD_AUTHOR', this.author);
                    });
            } else {
                // clear previous form data
                this.$store.commit('LOAD_AUTHOR', {});
            }
        },

        // confirm leaving page if the form is dirty
        beforeRouteLeave (to, from, next) {
            if (this.$store.state.dirty) {
                // could use the style below, but better to be clear than clever
                //next(window.confirm('You have unsaved changes. Are you sure you want to leave this page?'));

                if (!window.confirm('You have unsaved changes. Are you sure you want to leave this page?')) {
                    // stay on this page then
                    next(false);
                    return;
                }
            }

            // ok to leave
            // reset dirty state
            this.$store.commit('SET_DIRTY', false);
            next();
        },

        methods: {
            //// Helper/utility functions
            redirect() {
                // redirect to authors page after save
                this.$router.push({name: 'authorlist'});
            },

            authorFormIsValid() {
                let formIsValid = true;
                const errors = this.$store.state.errors || {};

                if (!this.$store.state.author.firstName || this.$store.state.author.firstName.length < 3) {
                    errors.firstName = 'First name must be at least 3 characters.';
                    formIsValid = false;
                } else {
                    delete errors['firstName'];
                }

                if (!this.$store.state.author.lastName || this.$store.state.author.lastName.length < 3) {
                    errors.lastName = 'Last name must be at least 3 characters.';
                    formIsValid = false;
                } else {
                    delete errors['lastName'];
                }

                this.$store.dispatch('UPDATE_ERRORS', {errors});
                return formIsValid;
            },


            //// Form handlers
            handleUpdateAuthorState(event) {
                const field = event.target.name;
                const author = this.$store.state.author;
                author[field] = event.target.value;

                // TODO keep a copy of the original data and actually compare changes instead of just assuming it changed
                // if user changes it and then edits it back to normal the form will still think it's dirty

                // mark state as dirty so we can trigger a leave page handler
                this.$store.commit('SET_DIRTY', true);
                this.$store.commit('SET_AUTHOR', {author: author});
                return this.authorFormIsValid();
            },

            handleSaveAuthor() {
                if (!this.authorFormIsValid()) {
                    return;
                }

                this.$store.dispatch('SAVE_AUTHOR', this.$store.state.author)
                    .then(() => {
                        this.redirect();
                    })
                    .catch(error => {
                        toastr.error(error);
                    });
            },

            handleDeleteAuthor(event) {
                let result = window.confirm('Are you sure you want to delete this author?');
                if (result) {
                    this.$store.dispatch('DELETE_AUTHOR', this.$store.state.author)
                        .then(author => {
                            this.redirect();
                        })
                        .catch(error => {
                            toastr.error(error);
                        });
                }
            }
        },

        computed: {
            // mix the getters into computed with object spread operator
            ...mapState([
                'saving',
                'deleting',
                'errors',
                'author'
            ])
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
