<template>
    <div class="submit-data-container">
      <template v-if="service">
          <div class="panel nav"><NavBar page="landingpage"/></div>
          <div class="content-container">
              <div style="display: flex;justify-content: space-between; align-items: baseline;">
                <h2 class="project-title">Register</h2><!-- <span>Already have an account? Please <a href="">Log in</a></span> -->
              </div>
              <span style="display: block; border-bottom: 1px solid rgba(100, 102, 100, 0.4);margin-bottom: 30px;"> </span>
              <!-- <Alert banner type="warning" style="margin-bottom: 20px">Registry Service is down. We will fix it soon!</Alert> -->
              <Alert v-if="disableForm" type="warning" show-icon>
                  Registry Service is down...
                  <template slot="desc">
                  We will fix it soon and sorry about any inconvenience.
                  </template>
              </Alert>
              <Form class="signUpForm" ref="formInlineSignUp" :model="formInlineSignUp" :rules="ruleInlineSignUp">
                <FormItem prop="email" label="Email">
                  <Input type="text" :disabled="disableForm" v-model="formInlineSignUp.email" placeholder="">
                  </Input>
                </FormItem>
                <FormItem prop="title" label="Title">
                  <Select :disabled="disableForm" v-model="formInlineSignUp.title">
                      <Option v-for="item in titleList" :value="item.value">{{item.label}}</Option>
                  </Select>
                </FormItem>
                <FormItem prop="firstname" label="First name">
                  <Input type="text" :disabled="disableForm" v-model="formInlineSignUp.firstname" placeholder="">
                  </Input>
                </FormItem>
                <FormItem prop="lastname" label="Last name">
                  <Input type="text" :disabled="disableForm" v-model="formInlineSignUp.lastname" placeholder="">
                  </Input>
                </FormItem>
                <FormItem prop="affiliation" label="Affiliation">
                  <Input type="textarea" :autosize="{minRows: 2,maxRows: 3}" :disabled="disableForm" v-model="formInlineSignUp.affiliation" placeholder="">
                  </Input>
                </FormItem>
                <FormItem prop="country" label="Country">
                  <Select :disabled="disableForm" v-model="formInlineSignUp.country">
                      <Option v-for="item in countryList" :value="item.value">{{item.label}}</Option>
                  </Select>
                </FormItem>
                <FormItem prop="orcid" label="ORCID">
                  <Input type="text" :disabled="disableForm" v-model="formInlineSignUp.orcid" placeholder="">
                  </Input>
                </FormItem>
                <FormItem prop="terms" label="Terms of Usage" >
                    <Checkbox :disabled="disableForm" v-model="formInlineSignUp.terms"><a class="privacy-action" @click="openTerms">Privacy notice</a></Checkbox>
                </FormItem>
                <FormItem>
                  <Button class="signupButton" :disabled="disableForm" type="primary" @click="signup('formInlineSignUp')" long>Sign Up</Button>
                </FormItem>
              </Form>
          </div>
      </template>
      <template v-else>
        <DefaultErrorPage err="Register Service Down"/>
      </template>
    </div>
</template>
<script>
    import NavBar from '@/components/Nav'
    import DefaultErrorPage from '@/components/DefaultErrorPage'
    import store from "@/store.js"
    export default {
        data () {
            return {
                signupAPI: this.$store.state.basePrivateURL + '/user/register',
                formInlineSignUp:{
                  email:'',
                  title:'',
                  firstname:'',
                  lastname:'',
                  affiliation:'',
                  country:'',
                  orcid:'',
                  terms:false,
                },
                ruleInlineSignUp:{
                  email: [
                    { required: true, type:'email', message: 'Please input email', trigger: 'blur' }
                  ],
                  title: [
                    { required: true, message: 'Please input title', trigger: 'blur' }
                  ],
                  firstname: [
                    { required: true, message: 'Please input firstname', trigger: 'blur' }
                  ],
                  lastname: [
                    { required: true, message: 'Please input lastname', trigger: 'blur' }
                  ],
                  affiliation: [
                    { required: true, message: 'Please input affiliation', trigger: 'blur' }
                  ],
                  country: [
                    { required: true, message: 'Please input country', trigger: 'blur' }
                  ],
                  orcid: [
                    { required: false, message: 'Please input orcid', trigger: 'blur' }
                  ],
                  terms:[
                    { required: true, type:'enum', enum: ["true"], transform: value => value.toString(), message: 'Please check Terms and Conditions' }
                  ],
                },
                titleList:[
                  {
                    label:'Mr',
                    value:'Mr'
                  },
                  {
                    label:'Ms',
                    value:'Ms'
                  },
                  {
                    label:'Miss',
                    value:'Miss'
                  },
                  {
                    label:'Mrs',
                    value:'Mrs'
                  },
                  {
                    label:'Dr',
                    value:'Dr'
                  },
                  {
                    label:'Professor',
                    value:'Professor'
                  },
                ],
                countryList:[],
                countryListURL: this.$store.state.baseURL + '/country/index.csv',
                service:true,
                disableForm:false,
            }
        },
        components: {
            NavBar,
            DefaultErrorPage
        },
        methods:{
            signup(name){
              this.$refs[name].validate((valid) => {
                  if (!valid) {
                    this.$Message.error({ content: 'Fill all required items',duration:3 });
                    return
                  }
                  this.$Spin.show({
                    render: (h) => {
                      return h('div', [
                        h('Icon', {
                          'class': 'demo-spin-icon-load',
                          props: {
                            type: 'load-b',
                            size: 18
                          }
                        }),
                        h('div', 'Please wait...')
                      ])
                    }
                  });
                  let query = {};
                  query.UserSummary = {
                    acceptedTermsOfUse: this.formInlineSignUp.terms,
                    affiliation: this.formInlineSignUp.affiliation,
                    country: this.formInlineSignUp.country,
                    email: this.formInlineSignUp.email,
                    firstName: this.formInlineSignUp.firstname,
                    lastName: this.formInlineSignUp.lastname,
                    orcid: this.formInlineSignUp.orcid,
                    title: this.formInlineSignUp.title,
                  }
                  this.$http
                        .post(this.signupAPI,query)
                        .then(function(res){
                              this.signUpModalBool=false;
                              this.$Message.success({ content: 'Sign Up Success! Please Check your email.', duration:10, closable: true })
                              this.$Spin.hide()
                              this.$refs[name].resetFields();
                              this.$router.push({name:'landingpage'});
                        },function(err){
                          this.$Spin.hide()
                          this.$Message.error({ 
                            top: 500, 
                            content: err.bodyText ? err.bodyText : "Error! Please email: pride-support@ebi.ac.uk",
                            duration:10,  
                            closable: true
                          });
                        });
              })
            },
            openTerms(){
              window.open('https://www.ebi.ac.uk/data-protection/privacy-notice/pride-new');
            },
            queryCountryList(){
                this.$http
                  .get(this.countryListURL)
                  .then(function(res){
                      this.countryList = [];
                      let json = JSON.parse(this.csvJSON(res.body)) 
                      for(let i=0; i < json.length-1; i++){
                          json[i].value = json[i].value.replace(/^"(.*)"$/, '$1');
                          let item = {}
                          item.label=json[i].value
                          item.value=json[i].value
                          this.countryList.push(item);
                      }
                  },function(err){

                  });
            },
            csvJSON(csv){
              var lines=csv.split("\n");
              var result = [];
              var headers=lines[0].split(",");
              for(var i=1;i<lines.length;i++){
                  var obj = {};
                  var currentline=lines[i].split(",");
                  for(var j=0;j<headers.length;j++){
                      obj[headers[j]] = currentline[j];
                  }
                  result.push(obj);
              }
              return JSON.stringify(result); //JSON
            },
        },
        mounted:function(){
          if(this.service){
            this.$refs['formInlineSignUp'].resetFields();
            this.queryCountryList();
          }  
        },
    }
</script>
<style scoped>
    .content-container{
        margin: 0 auto;
        padding: 20px 125px;
    }
    .project-title{
      color:rgb(100, 102, 100);
      margin: 20px 0 5px 0;
    }
    .privacy-action{
      color:#444;
    }
    .privacy-action:hover{
      color:#5bc0be;
    }
    @media (min-width: 768px) { 
        .content-container{
            width: 750px;
        }
    }
    @media (min-width: 992px) { 
        .content-container{
            width: 970px;
        }
    }
</style>
<style>

</style>
