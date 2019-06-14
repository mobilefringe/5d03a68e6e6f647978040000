<template>
    <div> <!-- without an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
        <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_header_background" v-if="pageBanner" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
                    <div class="main_container">
                        <h2>Contact Us</h2>
                    </div>
                </div>
                <div class="main_container margin_30">
                    <div class="details_row">
                        <div class="details_col_3 hidden_phone">
                            <img class="img_max" src="//codecloud.cdn.speedyrails.net/sites/5afdda1b6e6f643692370000/image/jpeg/1527708236000/Casas Adobes - Aerial 03.jpg" alt="" />    
                        </div>
                        <div class="details_col_9">
                            <div class="row">
                                <div class="col-md-12">
                                    <div class="inside_page_header">Get in Touch</div>
                                    <p>We value your feedback. Send us your comments or questions by completing the contact form below.</p>    
                                </div>
                                <form class="form-horizontal clearfix" action="form-submit" v-on:submit.prevent="validateBeforeSubmit">
                                    <div class="col-xs-12 col-md-6 margin_20" :class="{'has-error': errors.has('name')}">
                                        <label for="name">Name</label>
                                        <input v-model="form_data.name" v-validate="'required|alpha_spaces'" class="form-control" :class="{'input': true}" name="name" id="name" type="text" data-vv-delay="1000">
                                        <span v-show="errors.has('name')" class="form-control-feedback">{{ errors.first('name') }}</span>
                                    </div>
                                    <div class="col-xs-12 col-md-6 margin_20" :class="{'has-error': errors.has('email')}">
                                        <label for="email">Email</label>
                                        <input v-model="form_data.email" id="email" v-validate="'required|email'" class="form-control" :class="{'input': true}" name="email" type="email" data-vv-delay="1000">
                                        <span v-show="errors.has('email')" class="form-control-feedback">{{ errors.first('email') }}</span>
                                    </div>
                                    <div class="col-xs-12 margin_20" :class="{'has-error': errors.has('subject')}">
                                        <label for="subject">Subject</label>
                                        <input v-model="form_data.subject" id="subject" v-validate="'required:true'" class="form-control" :class="{'input': true}" name="subject" type="text" data-vv-delay="1000">
                                        <span v-show="errors.has('subject')" class="form-control-feedback">{{ errors.first('subject') }}</span>
                                    </div>
                                    <div class="col-xs-12 margin_20" :class="{'has-error': errors.has('message')}">
                                        <label for="message">Message</label>
                                        <textarea v-model="form_data.message" id="message" v-validate="'required:true'" class="form-control" :class="{'input': true}" name="message" type="text" data-vv-delay="1000"></textarea>
                                        <span v-show="errors.has('message')" class="form-control-feedback">{{ errors.first('message') }}</span>
                                    </div>
                                    <div class="col-xs-12">
                                        <button class="animated_btn" type="submit" :disabled="formSuccess">
                                            Send <i class="fa fa-angle-right" aria-hidden="true"></i>
                                        </button>
                                    </div>
                                </form>

                                <div id="send_contact_success" class="alert alert-success" role="alert" v-show="formSuccess">
                                    <span class="glyphicon glyphicon-ok" aria-hidden="true"></span>
                                    <span class="sr-only">Success</span>
                                    Thank you for contacting us. A member from our team will contact you shortly.
                                </div>
                                <div id="send_contact_error" class="alert alert-danger" role="alert" v-show="formError">
                                    <span class="glyphicon glyphicon-exclamation-sign" aria-hidden="true"></span>
                                    <span class="sr-only">Error:</span>
                                    There was an error when trying to submit your request. Please try again later.
                                </div>
                            </div>
                            <div class="row">
                                <div class="col-md-12">
                                    <hr>
                                    <div class="contact_page_body" v-html="currentPage.body"></div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>
<script>
    define(["Vue", "vuex", "vee-validate"], function (Vue, Vuex, VeeValidate) {
        Vue.use(VeeValidate);
        return Vue.component("contact-component", {
            template: template, // the variable template will be injected
            data: function data() {
                return {
                    dataLoaded: false,
                    pageBanner: "",
                    currentPage: null,
                    form_data: {},
                    loginPending: null,
                    formSuccess: false,
                    formError: false,
                    time: new Date()
                }
            },
            created() {
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Newsletter Banner');
                    if(temp_repo) {
                        this.pageBanner = temp_repo.images[0];
                    }
                    this.currentPage = response[1].data;
                    console.log(this.currentPage)
                    this.dataLoaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'findRepoByName'
                ])
            },
            methods: {
                loadData: async function () {
                    try {
                        this.property.mm_host = this.property.mm_host.replace("http:", "");
                        let results = await Promise.all([this.$store.dispatch("getData", "repos"), this.$store.dispatch('LOAD_PAGE_DATA', {url: this.property.mm_host + "/pages/casas-contact-us.json"})]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                },
                validateBeforeSubmit() {
                    this.$validator.validateAll().then((result) => {
                        if (result) {
                            let errors = this.errors;
                            send_data = {};
                            send_data.form_data = JSON.stringify(this.serializeObject(this.form_data));
                            this.$store.dispatch("CONTACT_US", send_data).then(res => {
                                this.formSuccess = true;
                            }).catch(error => {
                                try {
                                    if (error.response.status == 401) {
                                        console.log("Data load error: " + error.message);
                                        this.formError = true;
                                    } else {
                                        console.log("Data load error: " + error.message);
                                        this.formError = true;
                                    }
                                } catch (e) {
                                    console.log("Data load error: " + error.message);
                                    this.formError = true;
                                }
                            })
                        }
                    })
                },
                serializeObject(obj) {
                    var newObj = [];
                    _.forEach(obj, function (value, key) {
                        var tempVal = {};
                        tempVal.name = key;
                        tempVal.value = value;
                        newObj.push(tempVal);
                    });
                    return newObj;
                }
            }
        });
    });
</script>