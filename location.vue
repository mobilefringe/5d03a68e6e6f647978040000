<template>
    <div> <!-- for some reason if you do not put an outer container div this component template will not render -->
        <loading-spinner v-if="!dataLoaded"></loading-spinner>
       <transition name="fade">
            <div v-if="dataLoaded" v-cloak>
                <div class="inside_header_background" v-if="pageBanner" v-bind:style="{ backgroundImage: 'url(' + pageBanner.image_url + ')' }">
                    <div class="main_container">
                        <div class="page_container">
                            <h2>Location</h2>
                        </div>
                    </div>
                </div>
                <div class="main_container margin_30">
                    <div class="details_row">
                        <div class="details_col_3">
                            <!--<h3 class="inside_page_title">Office Phone Number</h3>-->
                            <!--<a class="inside_page_link" :href="'tel:' + property.contact_phone">{{ property.contact_phone }}</a>-->
                            <h3 class="inside_page_title">Address</h3>
                            <p class="inside_page_link">
                                {{ property.address1 }} <br>
                                {{ property.city }}, {{ property.province_state }} <br>
                                {{ property.postal_code }}
                            </p>
                            <a class="animated_btn" :href="siteInfo.googleMapsURL" target="_blank">Driving Direction</a>   
                        </div>
                        <div class="details_col_9">
                            <iframe title="Map" width="100%" height="400px" frameborder="0" scrolling="no" marginheight="0" marginwidth="0"  :src="'https://maps.google.nl/maps?q=' + getPropertyAddress + '&amp;hl=en&amp;ie=UTF8&amp;t=v&amp;hnear=' + getPropertyAddress + '&amp;z=16&amp;output=embed'"></iframe>
                            <hr>
                            <div class="page_content" v-if="currentPage" v-html="currentPage.body"></div>
                        </div>
                    </div>
                </div>
            </div>
        </transition>
    </div>
</template>

<script>
	define(["Vue", "vuex", "json!site.json"], function(Vue, Vuex, site) {
		return Vue.component("location-component", {
            template: template, // the variable template will be injected
            data: function () {
                return {
                    dataLoaded: false,
                    pageBanner: "",
                    currentPage: null,
                    siteInfo: site
                }
            },
            created() {
                this.loadData().then(response => {
                    var temp_repo = this.findRepoByName('Location Banner');
                    if(temp_repo) {
                        this.pageBanner = temp_repo.images[0];
                    }
                    
                    this.currentPage = response[1].data;
                    this.dataLoaded = true;
                });
            },
            computed: {
                ...Vuex.mapGetters([
                    'property',
                    'findRepoByName'
                ]),
                getPropertyAddress() {
                    return this.property.name + ' ' + this.property.address1 + ' ' + this.property.city + ' ' + this.property.country + ' ' +this.property.province_state + ' ' + this.property.province_state
                }
            },
            methods: {
                loadData: async function () {
                    try {
                        this.property.mm_host = this.property.mm_host.replace("http:", "");
                        let results = await Promise.all([this.$store.dispatch("getData", "repos"), this.$store.dispatch('LOAD_PAGE_DATA', {url: this.property.mm_host + "/pages/casas-location-map.json"})]);
                        return results;
                    } catch (e) {
                        console.log("Error loading data: " + e.message);
                    }
                }
            }
        });
	});
</script>