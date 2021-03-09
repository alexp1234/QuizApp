<template>
  <div class="container-fluid mt-4">
      <h1>Practice Quiz: Question {{currentPage + 1}} of 5</h1>
      <div v-if="!hasSubmitted || inReviewMode">
      <div class="card shadow col-md-6 offset-md-3" style="padding-bottom:20px;" >   
          <div class="mb-4 mt-4">
                {{currentField.Sentence}}
          </div>
          <span class="text-danger" v-if="displayErrorMessage">You must make a selection.</span>
          <div v-for="answer in questionAnswers" :key="answer" class="container-fluid" style="margin-left:-5%;">
              <div class="form-check">
                   <label class="col-md-2 form-check-label" :for="answer"><span class="text-danger" v-if="inReviewMode && allUserAnswers[currentPage].userChoice == answer">Your Choice&nbsp;</span>
                   <span class="text-success" v-if="currentField['Subordination Code'] == answer && inReviewMode">Correct&nbsp;</span>{{answer}}</label>              
                   <input @click="setAnswer(answer, currentField.Sentence, currentField['Subordination Code'])" :id="answer" name="answers" class="col-md-1 form-check-input" type="radio" :value="answer" :disabled="inReviewMode"/>
              </div>
          </div>
          
      </div>
      <div class="row mt-4 mb-4">
          <div class="col-md-6 offset-md-3">
           <button @click="previousQuestion()" v-if="currentPage > 0" class="btn btn-secondary" style="max-width:44%;float:left;">Back</button>
          <button v-if="currentPage < 4" @click="nextQuestion()" class="btn btn-primary" style="max-width:44%; float:right;">Next Question</button>
          <button @click="submit()" v-else-if="currentPage == 4 && !inReviewMode" style="float:right; max-width:44%;" class="btn btn-success">Submit</button>
          <button @click="reset()" v-else style="float:right; max-width:44%;" class="btn btn-success">Reset</button>

          </div>
      </div>
      </div>
      <div v-else>
          <div class="card shadow col-md-6 offset-md-3" >
              <h3 class="mt-4 mb-4">Your Score: <strong>{{getFinalScore}}%</strong></h3>
              <button @click="goToReview()" class="btn btn-primary mb-2 col-md-4 offset-md-4">Review Correct Answers</button>
          </div>
      </div>
  </div>
</template>

<script>
import axios from 'axios';
export default {
    data(){
        return{
            inReviewMode: false,
            currentPage: 0,
            hasSubmitted: false,
            displayErrorMessage: false,
            //userAnswers: {1: userAnswer:"", c},
            userAnswer:{},
            allUserAnswers:[],
            currentField: {},
            questions: [],
            randomNumbers: [],
            questionAnswers: ["[ec]", "[em]", "[ep]", "[re]", "[ri]", "[s]", "[q]",  "[ec/s]", "[ec/ri]" ]
        }
    },
    computed:{
        getFinalScore: function(){
            var correctCount = 0;
            var incorrectCount = 0;
            for(var i = 0; i < this.allUserAnswers.length; i++){
                if(this.allUserAnswers[i].userChoice == this.allUserAnswers[i].correctAnswer){
                    correctCount += 1;
                }
                else{
                    incorrectCount +=1;
                }
            }
            return (correctCount/ (correctCount + incorrectCount)) * 100;
        }
    },
    methods:{
        nextQuestion(){
            if(this.currentPage < 4){
            if(this.allUserAnswers[this.currentPage]){
                   this.currentPage += 1;
             this.displayErrorMessage = false;
             this.currentField = this.questions[this.currentPage].fields;
             var ele = document.getElementsByName("answers");
             for(var i=0;i<ele.length;i++)
                ele[i].checked = false
            
            if(this.allUserAnswers[this.currentPage]){
                    var choice = this.allUserAnswers[this.currentPage].userChoice;
                    document.getElementById(choice).click();
            }
            window.scrollTo(0,0)

            }
            else{
                this.displayErrorMessage = true;
            }     
            }              
        },
        reset(){
           window.location.href = "https://testquizsite.netlify.app";
        },
        goToReview(){
            this.currentPage = 0;
            this.currentField = this.questions[this.currentPage].fields;
            this.inReviewMode = true;
            
        },
        previousQuestion(){
            // need logic to set input to what user chose
            if(this.currentPage > 0){
                this.currentPage -= 1;
                this.currentField = this.questions[this.currentPage].fields;               
            }
            var choice = this.allUserAnswers[this.currentPage].userChoice;
            document.getElementById(choice).click();
            window.scrollTo(0,0)
        },
        setAnswer(userChoice, sentence, correctAnswer){
            for(var i = 0; i < this.allUserAnswers.length; i++){
                if(this.allUserAnswers[i].questionNumber == this.currentPage){
                    this.allUserAnswers[i].userChoice = userChoice;
                    this.allUserAnswers[i].sentence = sentence;
                    this.allUserAnswers[i].correctAnswer = correctAnswer;
                    return true;
                }
            }
           var userAnswer = {}
           userAnswer.questionNumber = this.currentPage
           userAnswer.userChoice = userChoice;
           userAnswer.sentence = sentence;
           userAnswer.correctAnswer = correctAnswer;      
           this.allUserAnswers.push(userAnswer);   
        },
        submit(){
            if(this.allUserAnswers[4]){
                this.displayErrorMessage = false;
                this.hasSubmitted = true;
            }
            else{
                this.displayErrorMessage = true;
            }
            
        }
    },
    created(){
        axios.get("https://api.airtable.com/v0/appBUy7ywNbFhXtfg/Single%20Sentences%20Database", {
             headers: {
            'Authorization': 'Bearer ' + 'keyun1qszEWlZOaFb'
             }
        })
        .then((response) => {
            //this.allQuestions = response.data.records;                          
                 for(var i = 0; i < 5; i++){
                        var randomNumber = Math.floor(Math.random() * response.data.records.length)
                        this.randomNumbers.push(randomNumber)
                 }
                for(var j = 0; j < response.data.records.length; j++){                
                if(this.randomNumbers.includes(j)){
                    this.questions.push(response.data.records[j])                   
                }                          
            }
            this.currentField = this.questions[0].fields            
        })
    }
    



}
</script>

<style>

</style>