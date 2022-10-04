<template>
    <div class="container">
        <div class = "row">
            <div class="header-text">Расчитывайте стоимость автомобиля в лизинг</div>
            <div class="range-container">
                <div class="price"><span>Желаемая сумма кредита</span></div>
                <div class="price-range">
                    <div>
                        <input type="text" :value="this.PriceStyle(this.price)" name="this.price" v-on:input="Range(DefaultStyle($event.target.value),true)" />
                        <span>&#x20bd;</span>
                    </div>
                    <input id="price" type="range" @change="LastColor($event.target.id)" v-on:input="Range($event.target.value)" :value="this.price" min="1000000" max="4000000">
                </div>
            </div>
            <div class="range-container">
                <div class="price"><span>Первоначальный взнос</span></div>
                <div class="price-range">
                    <div>
                        <input type="text" :value="this.PriceStyle((this.price*this.percent)/100)"  v-on:input="Percent(DefaultStyle($event.target.value),true)"/>
                        <span class="percent"><div>{{this.percent}}&#x25;</div></span>
                    </div>
                    <input id="percent" type="range" @change="LastColor($event.target.id)"  v-on:input="Percent($event.target.value)" :value="this.percent" min="10" max="60">
                </div>
            </div>
            <div class="range-container">
                <div class="price"><span>Срок лизинга</span></div>
                <div class="price-range">
                    <div>
                        <input type="text" :value="this.month"  v-on:input="Month(DefaultStyle($event.target.value),true)" />
                        <span>мес.</span>
                    </div>
                    <input id="month" type="range" @change="LastColor($event.target.id)"  v-on:input="Month($event.target.value)" :value="this.month" min="1" max="60">
                </div>
            </div>
            <div class="range-container col-5" >
                <div class="row">
                        <p id="pay">Сумма договора лизинга</p>
                        <span>{{ Sumcontract()}} &#x20bd;</span>
                    </div>
            </div>
            <div class="range-container col-5">
                <div class="row">
                        <p id="pay">Ежемесячный платеж от</p>
                        <span>{{MonthPay().toString().replace(/\B(?=(\d{3})+(?!\d))/g, " ")}} &#x20bd;</span>
                    </div>
            </div>
            <div class="range-container col-5">
                <div id="btn">
                    <button id="button" @click="Response()"><p id="text">Оставить заявку</p><div class="priloader" id="loader"></div></button>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
    
    export default {
        name: "CalculatorAvto",
        data(){
            return{
                price:1000000,
                percent:10,
                firstpay: this.price/ this.percent,
                month:1
            }
        },
        methods:{
            CheckButton(price){
                if(price == 0 || this.percent == 0 || this.month ==0){
                    document.getElementById("button").style.opacity = "0.4"
                    document.getElementById("button").style.cursor = "not-allowed"
                } else{
                    document.getElementById("button").style.opacity = "1"
                    document.getElementById("button").style.cursor = "default"
                }
            },
            Response(){
                const information = {"car_coast": this.price,
                                "initail_payment": (this.price*this.percent)/100,
                                "initail_payment_percent": this.percent,
                                "lease_term": this.month,
                                "total_sum": this.DefaultStyle(this.Sumcontract()),
                                "monthly_payment_from": this.MonthPay(),
                            };
                document.getElementById("loader").classList.add("loader")
                document.getElementById("text").style.display = "none"
                document.getElementById("btn").classList.add("btn_disabled")
                fetch('https://eoj3r7f3r4ef6v4.m.pipedream.net',{
                    method:"POST",
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify(information)
                })
                .then(res => res.json())
                        .then((result)=>{console.log(result)
                            document.getElementById("loader").classList.remove("loader")
                            document.getElementById("text").style.display = "block"
                            document.getElementById("btn").classList.remove("btn_disabled")
                        })  
            },
            PriceStyle(x){
            const stylePrice = x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, " ")
            return stylePrice
            },
            DefaultStyle(x){
               return x.split(' ').join('')
            },
            OpacityStyle(id,x){
                var listId = ["price","percent","month"]
                for(let i = 0;i<listId.length;i++){
                    if(listId[i]!==id){
                        let child = document.getElementById(listId[i]);
                        child.parentElement.closest('div').style.opacity = x;
                    }
                }
            },
            RangeStyle(id,val,max,min,type){
                if(val<min){
                    val = min
                } else if(val>max){
                    val = max
                }
                document.getElementById(id).style.backgroundSize = `${(val-min)*100/(max-min)}% 100%`;
                const child = document.getElementById(id);
                if(type == undefined){
                this.OpacityStyle(id,"0.4")
                child.parentElement.closest('div').style.backgroundColor = "white";
                }
            },
            LastColor(id){
                this.OpacityStyle(id,"")
                const child = document.getElementById(id);
                child.parentElement.closest('div').style.backgroundColor = "#E5E5E5";
            },
             Range(price,type){
                const min = 1000000
                const max = 4000000
                if(price>=1000000 && price<=max){
                this.price = price
                } 
                this.RangeStyle("price",price,4000000,min,type)
                this.CheckButton(price)
            },
            Percent(percent,type){
                if(type == true){
                    this.percent = (percent*100)/this.price
                } else{
                this.percent = percent
                this.RangeStyle("percent",percent,60,10,type)
                }
                this.CheckButton()
            },
            Month(month,type){
                if(month==""){
                    this.month = 0
                } else {
                this.month = month
                this.RangeStyle("month",month,60,1,type)
                }
                this.CheckButton()
            },
            MonthPay(){
                const monthpay = (this.price - (this.price/ this.percent))*((0.035*Math.pow((1+0.035),this.month))/ (Math.pow((1 + 0.035), this.month) - 1))
                return Math.ceil(monthpay)
            },
            Sumcontract(){
                const sum  = (this.price/ this.percent)+this.month*this.MonthPay()
                return this.PriceStyle(Math.ceil(sum))
            }
            
        }
    }
</script>
<style data="test" type="text/css">
    #text{
        margin: 0px;
    }
    input {outline:none;
        background: none;
        outline: none;
        border: none;
    }
     .price-range>div>input{
        font-family: Gilroy, Arial, sans-serif;
        font-size: 22px;
    line-height: 20px;
      font-weight: bold;
      color: rgb(53, 62, 71);
      width: 90%;
    }
    .price-range>div>span{
        font-family: Gilroy, Arial, sans-serif;
        font-size: 22px;
    line-height: 20px;
      font-weight: bold;
      color: rgb(53, 62, 71);
    }
    .row>span{
    font-family: Gilroy, Arial, sans-serif;
    font-size: 22px;
    line-height: 20px;
    font-weight: bold;
    color: rgb(53, 62, 71);
    }
#pay{
    width: 100%;
    font-family: OpenSans, Arial, sans-serif;
    font-weight: bold;
    color: rgb(53, 62, 71);
}
div>p{
    margin-bottom: 0px;
}
.priloader{
    display: block;
    margin: 0 auto;
}
    button {
    width: 100%;
    margin-top: 15px;
    background-color: rgb(241, 154, 60);
    border: none;
    color: white;
    padding: 15px 32px;
    text-align: center;
    text-decoration: none;
    display: block;
    font-family: Gilroy, Arial, sans-serif;
    font-style: normal;
    font-weight: 900;
    font-size: 22px;
    line-height: 20px;
    border-radius: 40px;
}

button:hover{
    background-color: black;
};
button:hover:after{

}
button:active {
  background-color: #575757;
}

  
.row>p{
font-family: OpenSans, Arial, sans-serif;
font-weight: bold;
font-size: 14px;
line-height: 19.04px;
color: rgb(53, 62, 71);
}

.percent>div{
    position: absolute;
    right: 0px;
    bottom: -10px;
    padding: 11px;
    background-color:#9c989873 ;
    border-radius: 10px;
    font-family: Gilroy, Arial, sans-serif;
    font-weight: bold;
    font-size: 22px;
    line-height: 20px;
}
.percent{
    position: relative;
}

div.range-container {
    margin-top: 10px;
  flex: 0 0 auto;
  width: 100%;
}
.row{
    margin-top: 30px;
}
.price {
    font-weight: bold;
    margin-bottom: 10px;
}
.loader {
    border: 3px solid #ffffff;
  border-radius: 50%;
  border-top: 3px solid rgba(100, 100, 100, 0);
  width: 20px;
  height: 20px;
  -webkit-animation: spin 2s linear infinite; /* Safari */
  animation: spin 2s linear infinite;
  cursor: not-allowed;
}
.btn_disabled:hover #button{
    background-color: rgb(241, 154, 60) !important;
    cursor: not-allowed;
}
@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
.price-range{
    border:2px solid rgba(243, 243, 244, 1);
    background: #E5E5E5;
    padding: 20px;
    padding-bottom: 0px;
    border-radius: 20px;
    display: flex;
    flex-direction: column;
}
.price>span{
    display: flex;
    font-weight: normal;
    font-family: OpenSans, Arial, sans-serif;
    font-size: 14px;
    line-height: 19.04px;
}
.price-range>div{
    display: flex;
    justify-content: space-between;
    padding-bottom: 18px;
}
input[type="range"] {
    margin: 0;
    -webkit-appearance: none;
    width: 100%;
    height: 2px;
    background-image: linear-gradient(orange, orange);
    background-size: 0% 100%;
    background-color:#9c989873 ;
    background-repeat: no-repeat;
 }
 input[type="range"]:hover {
    height: 1px;
 }
 
input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    height: 15px;
    width: 15px;
    border-radius: 50%;
    background: orange;
    cursor: ew-resize;
    }
 
input[type="range"]::-moz-range-thumb {
    -webkit-appearance: none;
    height: 15px;
    width: 15px;
    border-radius: 50%;
    background: orange;
    cursor: ew-resize;
}
 
input[type="range"]::-ms-thumb {
    -webkit-appearance: none;
    height: 15px;
    width: 15px;
    border-radius: 50%;
    background: orange;
    cursor: ew-resize;
}
    .container{
        width: 90%;
        margin-right: auto;
        margin-left: auto;
    }

    .row{
        --bs-gutter-x: 1.5rem;
        --bs-gutter-y: 0;
        display: flex;
        flex-wrap: wrap;
        margin-top: calc(var(--bs-gutter-y) * -1);

    }
    .header-text{
        font-family: Gilroy, Arial, sans-serif;
        font-weight: 900;
        font-size: 34px;
        flex: 0 0 auto;
        width: 80%;
        line-height: 30.6px;
    }
    @font-face {
        font-family: "OpenSans";
        src: url("../../fontGilroy/Gilroy-Light.woff2") format("woff2"),
        url("../../fontGilroy/Gilroy-Light.woff") format("woff");
        font-weight: normal;
        font-style: normal;
        font-display: swap;
        }

        @font-face {
        font-family: "OpenSans";
        src: url("../../fontGilroy/Gilroy-Light.woff2") format("woff2"),
        url("../../fontGilroy/Gilroy-Light.woff") format("woff");
        font-weight: 400;
        font-style: normal;
        font-display: swap;
        }
        @font-face {
        font-family: "Gilroy";
        src: url("../../font/Nekst-Black.woff2") format("woff2"),
        url("../../font/Nekst-Black.woff") format("woff");
        font-weight: normal;
        font-style: normal;
        font-display: swap;
        }

        @font-face {
        font-family: "Gilroy";
        src: url("../../font/Nekst-Black.woff2") format("woff2"),
        url("../../font/Nekst-Black.woff") format("woff");
        font-weight: 900;
        font-style: normal;
        font-display: swap;
        }

@media screen and (min-width: 425px) and (max-width: 768px) {
    .col-5{
        width: 50% !important;
    }
    .header-text{
        size: 54px;
        line-height: 48.6px;
        font-weight: 900;
    }
}
@media screen and (min-width: 768px) and (max-width: 1025px) {
    .col-5{
        width: 50% !important;
    }
    .header-text{
        size: 54px;
        line-height: 48.6px;
        font-weight: 900;
    }
}
@media screen and (min-width: 1025px) and (max-width: 1083px) {
    div.range-container{
       width: 25%;
       margin-right: 32px;
    }
} 
@media screen and (min-width: 1083px) and (max-width: 1440px) {
    div.range-container{
       width: 25%;
       margin-right: 32px;
    }
} 
@media screen and (min-width: 1440px) and (max-width: 2560px) {
    div.range-container{
       width: 30%;
       margin-right: 32px;
    }
} 
@media screen and (min-width: 500px) and (max-width: 769px) {
    #displaySum>div{
        width: 50%;
    }
} 
</style>