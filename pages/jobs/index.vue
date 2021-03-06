<template>
  <div>
    <template v-if="!pageLoading">
      <section class="page-title pb-5">
        <div class="row">
          <div class="col-md-8">
            <h1 class="h4 mb-0 page-title">All Jobs</h1>
          </div>
          <div class="col-md-4 text-right">
            <!-- search field -->
            <div class="form-group">
              <div class="input-group">
                <input 
                  v-model="search" 
                  type="text" 
                  class="form-control fs-14"  
                  placeholder="filter by job name and location..."
                  @change="togglePagination" 
                >
              </div>
            </div>
          </div>
        </div>
      </section>
  
      <!-- latest jobs -->
      <section class="mb-5" v-if="allJobs !== null">
        
        <template v-if="paginatedJobs.length">
          <div class="row mb-5">
            <div class="col-md-3 col-sm-6 px-2 pb-4" v-for="job in paginatedJobs" :key="job._id">
              <div class="card job-card">
                <div class="card-body text-center">
                  <div class="d-flex justify-content-center">
                    <div class="avatar-parent-child">
                      <Logo />
                      <div class="position-absolute" style="left: 152px; top: 40px;"  v-if="job.status === 'Open'">
                        <small class="text-success"><i class="fas fa-circle"></i></small>
                      </div>
                      <div class="position-absolute" style="left: 152px; top: 40px;" v-if="job.status === 'Closed'">
                        <small class="text-danger"><i class="fas fa-circle"></i></small>
                      </div>
                    </div>
                  </div>
                  <nuxt-link :to="({path: `/jobs/${job._id}`})" class="d-block h6 mt-4 mb-1">{{ job.title }}</nuxt-link>
                  <span class="d-block text-sm text-muted mb-3 job-description">{{ job.description }}</span>
                </div>
                <div class="card-footer border-0 bg-white d-flex justify-content-between mb-2">
                  <div class="d-flex align-items-center">
                    <img src="~assets/images/location.svg" height="15" alt="">
                    <small class="text-dark pl-1">{{ job.location }}</small>
                  </div>
                  <small class="text-warning">&#36; {{ job.salary }}</small>
                </div>
              </div>
            </div>
          </div>
          <!-- pagination -->
          <client-only>
            <div class="row justify-content-center text-center" v-if="allJobs !== null">
              <div class="col-md-12 d-flex justify-content-center">
                <paginate
                  v-if="togglePagination" 
                  :page-count="pageCount"
                  :page-range="3"
                  :margin-pages="2"
                  :click-handler="clickCallback"
                  :prev-text="`<i class='fa fa-arrow-left fs-14'></i>`"
                  :next-text="`<i class='fa fa-arrow-right fs-14'></i>`"
                  :container-class="'pagination'"
                  :page-class="'page-link'"
                >
                </paginate>
              </div>
            </div>
          </client-only>
        </template>
        <template v-else>
          <div class="row justify-content-center">
            <div class="col-md-7 text-center">
              <img src="~/assets/images/illustration-4.png" alt="">
              <h4 class="h4">OOps! No Jobs </h4>
              <p class="text-muted">No jobs matches your search criteria, try using other keywords</p>
            </div>
          </div>
        </template>
      </section>
    </template>

    <template v-else>
      <page-loader />
    </template>
  </div>
</template>

<script>
  import Logo from '@/components/Logo'
  import PageLoader from '@/components/pageLoader'
  export default {
    layout: 'auth',
    components: {
      Logo,
      PageLoader
    },
    props: {
      size: {
        type: Number,
        required: false,
        default: 8
      }
    },
    async mounted() {
      if (this.allJobs === null) {
        this.getJobs()
      }
      else {
        this.pageLoading = false
      }
    },
    methods: {
      async getJobs() {
        this.pageLoading = true
        await this.$store.dispatch('dashboard/allJobs').then(res => {
          this.pageLoading = false
        }).catch(err => {
          this.pageLoading= false
        })
      },
      clickCallback(pageNum) {
        this.indexNumber = pageNum - 1
      },
      togglePagination () {
        if (this.search) {
          return false
        }   else {
          return true
        }
      },
    },
    computed: {
      allJobs() {
        return this.$store.state.dashboard.jobs
      },
      latestJobs() {
        return this.$store.getters['dashboard/latestJobs']
      },
      closedJobs() {
        return this.$store.getters['dashboard/closedJobs']
      },
      openJobs() {
        return this.$store.getters['dashboard/openJobs']
      },
      pageCount() {
        const l = this.allJobs.jobs.length
        const s = this.size
        return Math.ceil(l / s)
      },
      paginatedJobs() {
        const self = this
        const start = parseInt(self.indexNumber) * self.size
        const end = start + self.size
        if (self.search == "") {
          return self.allJobs.jobs.slice(start, end)
        }else {
          return self.allJobs.jobs.filter((job) => {
            return ((job.title.toLowerCase().match(self.search.toLowerCase()))  || (job.location.toLowerCase().match(self.search.toLowerCase())) )
          })
        }
      },
    },
    data() {
      return {
        indexNumber: 0,
        page: 1,
        search: '',
        pageLoading: true
      }
    },
  }

</script>

<style>
  .page-title {
    color: #152c5b;
    letter-spacing: -1px;
    margin-bottom: 3px;
  }

  .icon-shape {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    text-align: center;
    vertical-align: middle;
    border-radius: .375rem;
    width: 3rem;
    height: 3rem;
  }

  .all-jobs-link {
    font-size: 14px;
    color: #152c5b;
  }

  .all-jobs-link span {
    border-bottom: 1px dotted;
  }

  .job-description {
    display: -webkit-box !important;
    -webkit-box-orient: vertical;
    -webkit-line-clamp: 3;
    text-overflow: ellipsis;
    overflow: hidden;
  }

  .job-card {
    height: 100%;
    min-height: 300px;
    margin-bottom: 0;
  }

  .avatar-parent-child {
    height: 55px;
    width: 55px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 100%;
    background: #000;
    padding: 5%;
    text-align: center;
  }

  .avatar-parent-child svg {
    position: absolute;
    height: 30px;
    width: 30px;
  }

</style>
