# Index : [[EtcIndex]]
## Tag : #Study 
---
   
Udemy 한글자막 보는 법 참고
https://fomaios.tistory.com/entry/Udemy-%EC%8B%A4%EC%8B%9C%EA%B0%84-%ED%95%9C%EA%B8%80-%EC%9E%90%EB%A7%89%EC%9C%BC%EB%A1%9C-%EB%B3%B4%EB%8A%94-%EB%B0%A9%EB%B2%95featChrome

```
if(typeof window.i==='undefined'){clearInterval(window.i)}
else{
let lastText='';
function check(){
let toEl=document.querySelector('.well--container--2edq4 span');
let fromEl=document.querySelector('p[data-purpose="transcript-cue-active"] span');
let currentText=fromEl.innerText;
if(lastText!==currentText){toEl.innerText= currentText}lastText=fromEl.innerText}window.i=setInterval(check,200)
}
```


* Plugin : https://udemy.wjtb.co.kr/notice/id/1772

2023/11/16 이후로 작동안됨
다음 코드 사용
```javascript
if(typeof window.i==='undefined') {  
clearInterval(window.i)  
} else {  
let lastText='';  
function check() {  
let toEl=document.querySelector('.well--container--1cXrh span');  
let fromEl=document.querySelector('p[data-purpose="transcript-cue-active"] span');  
let currentText=fromEl.innerText;  
if(lastText!==currentText) {  
toEl.innerText= currentText  
}  
lastText=fromEl.innerText  
}  
window.i=setInterval(check,200)  
}
```