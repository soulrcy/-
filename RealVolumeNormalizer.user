// ==UserScript==
// @name         RealVolumeNormalizer v2.1
// @namespace    RVN
// @version      2.1
// @description  Automatic video volume normalization
// @match        *://*/*
// @grant        none
// @run-at       document-idle
// ==/UserScript==


(function(){

"use strict";


console.log("[RVN] v2.1 start");



// =================
// CONFIG
// =================

const CONFIG={


    enabled:true,


    minDb:-35,

    maxDb:-20,


    windowTime:5,


    adjustSpeed:0.05,


    minVolume:0.05,

    maxVolume:1


};




// =================
// STATE
// =================

let video=null;


let audioContext=null;

let source=null;

let analyser=null;


let buffer=null;


let connected=false;



let samples=[];


let currentDb=-100;





// =================
// Utils
// =================


function clamp(v,min,max){

    return Math.max(
        min,
        Math.min(
            max,
            v
        )
    );

}



function rmsToDb(rms){

    return 20 *
    Math.log10(
        Math.max(
            rms,
            0.00001
        )
    );

}





// =================
// Connect Audio
// =================


async function connectVideo(v){


    console.log(
        "[RVN] connecting",
        v
    );



    try{


        if(audioContext){

            try{

                await audioContext.close();

            }
            catch(e){}


        }




        let AC =
        window.AudioContext ||
        window.webkitAudioContext;



        audioContext =
            new AC();




        source =
        audioContext
        .createMediaElementSource(
            v
        );



        analyser =
        audioContext
        .createAnalyser();



        analyser.fftSize=2048;



        buffer =
        new Float32Array(
            analyser.fftSize
        );



        /*

        关键：

        只检测

        不修改声音


        */


        source.connect(
            analyser
        );


        analyser.connect(
            audioContext.destination
        );



        await audioContext.resume();



        video=v;


        connected=true;


        console.log(
            "[RVN] audio connected"
        );


    }
    catch(e){


        console.error(
            "[RVN] audio connect error",
            e
        );


    }



}




// =================
// Find Video
// =================


function findVideo(){


    let list =
    document.querySelectorAll(
        "video"
    );



    for(
        let v of list
    ){

        if(
            !v.paused &&
            !v.ended
        ){

            return v;

        }

    }



    return list[0];

}




function scanVideo(){


    let v =
    findVideo();



    if(
        v &&
        v!==video
    ){

        connectVideo(v);

    }


}




setInterval(
    scanVideo,
    1000
);
// =================
// Audio Analyse
// =================


function analyseAudio(){


    if(
        !connected ||
        !analyser ||
        !video
    ){

        return;

    }



    if(
        !CONFIG.enabled
    ){

        return;

    }




    analyser.getFloatTimeDomainData(
        buffer
    );



    let sum=0;



    for(
        let i=0;
        i<buffer.length;
        i++
    ){

        let v =
            buffer[i];


        sum +=
            v*v;

    }




    let rms =
        Math.sqrt(
            sum /
            buffer.length
        );



    let db =
        rmsToDb(
            rms
        );




    samples.push({

        time:
        Date.now(),

        db:db

    });



    cleanSamples();


    calculateAverage();



}




// =================
// Remove old data
// =================


function cleanSamples(){


    let now =
        Date.now();



    let time =
        CONFIG.windowTime *
        1000;



    samples =
    samples.filter(

        item =>

        now -
        item.time
        <=
        time

    );


}





// =================
// Average loudness
// =================


function calculateAverage(){


    if(
        samples.length===0
    ){

        return;

    }



    let total=0;



    for(
        let s of samples
    ){

        total +=
            s.db;

    }




    currentDb =
        total /
        samples.length;




    adjustVideoVolume();




    if(
        window.RVN_UpdateUI
    ){

        window.RVN_UpdateUI();

    }


}





// =================
// Volume Control
// =================


function adjustVideoVolume(){


    if(
        !video
    ){

        return;

    }



    let target =
        video.volume;



    /*

    低于最低值

    增加播放器音量

    */


    if(
        currentDb <
        CONFIG.minDb
    ){


        let diff =

            CONFIG.minDb -
            currentDb;



        /*

        防止一次跳太大

        */

        target +=
            diff /
            120;



    }




    /*

    高于最高值

    降低播放器音量

    */


    else if(
        currentDb >
        CONFIG.maxDb
    ){


        let diff =

            currentDb -
            CONFIG.maxDb;



        target -=
            diff /
            120;


    }
    else{


        return;

    }





    target =
        clamp(

            target,

            CONFIG.minVolume,

            CONFIG.maxVolume

        );





    /*

    平滑修改

    */


    let old =
        video.volume;



    let next =

        old +

        (
            target -
            old
        )
        *
        CONFIG.adjustSpeed;



    if(
        Math.abs(
            next-old
        )
        <
        0.005
    ){

        return;

    }




    video.volume =
        next;



}





// =================
// Main Loop
// =================


setInterval(
    analyseAudio,
    100
);
// =====================
// UI Style
// =====================


function createStyle(){


    let style =
    document.createElement(
        "style"
    );


    style.textContent = `


#rvn-panel{


position:fixed;

right:20px;

bottom:20px;

width:320px;

background:#151515;

color:white;

padding:14px;

border-radius:12px;

font-family:Arial;

font-size:14px;

z-index:2147483647;

box-shadow:0 5px 25px #000;


}



#rvn-panel.mini
.rvn-body{

display:none;

}



.rvn-head{

display:flex;

justify-content:space-between;

align-items:center;

font-weight:bold;

}



.rvn-row{

display:flex;

justify-content:space-between;

align-items:center;

margin:8px 0;

}



.rvn-btn{

background:#333;

color:white;

border:1px solid #666;

border-radius:5px;

padding:4px 10px;

cursor:pointer;

}



.rvn-input{

width:75px;

background:#222;

color:white;

border:1px solid #666;

text-align:center;

}



.rvn-status{

margin-top:10px;

background:#222;

padding:8px;

border-radius:6px;

text-align:center;

}


`;



    document.head.appendChild(
        style
    );

}




// =====================
// UI Variables
// =====================


let panel=null;


let dbText=null;

let volumeText=null;

let statusText=null;





// =====================
// Create UI
// =====================


function createUI(){


    panel =
    document.createElement(
        "div"
    );



    panel.id =
    "rvn-panel";



    panel.innerHTML = `


<div class="rvn-head">


<span>
🔊 RVN v2.1
</span>


<div>


<button
class="rvn-btn"
id="rvn-mini"
>
—
</button>


<button
class="rvn-btn"
id="rvn-close"
>
×
</button>


</div>


</div>



<div class="rvn-body">



<div class="rvn-row">

<span>
自动调节
</span>


<button
class="rvn-btn"
id="rvn-power"
>
ON
</button>


</div>



<div class="rvn-row">

<span>
当前响度
</span>


<span id="rvn-db">
--
</span>


</div>



<div class="rvn-row">

<span>
播放器音量
</span>


<span id="rvn-volume">
--
</span>


</div>



<div class="rvn-row">

<span>
最低 dB
</span>


<input
class="rvn-input"
id="rvn-min"
type="number"
>


</div>




<div class="rvn-row">

<span>
最高 dB
</span>


<input
class="rvn-input"
id="rvn-max"
type="number"
>


</div>




<div class="rvn-row">

<span>
检测时间(s)
</span>


<input
class="rvn-input"
id="rvn-window"
type="number"
>


</div>




<div class="rvn-status"
id="rvn-status"
>

等待视频

</div>



</div>


`;



    document.body.appendChild(
        panel
    );



    dbText =
    document.getElementById(
        "rvn-db"
    );



    volumeText =
    document.getElementById(
        "rvn-volume"
    );



    statusText =
    document.getElementById(
        "rvn-status"
    );



    document.getElementById(
        "rvn-min"
    ).value =
    CONFIG.minDb;



    document.getElementById(
        "rvn-max"
    ).value =
    CONFIG.maxDb;



    document.getElementById(
        "rvn-window"
    ).value =
    CONFIG.windowTime;



    bindUI();


}





// =====================
// UI Events
// =====================


function bindUI(){



    document.getElementById(
        "rvn-power"
    ).onclick=function(){


        CONFIG.enabled =
        !CONFIG.enabled;



        this.textContent =
        CONFIG.enabled ?
        "ON":
        "OFF";


    };






    document.getElementById(
        "rvn-mini"
    ).onclick=function(){


        panel.classList.toggle(
            "mini"
        );


    };






    document.getElementById(
        "rvn-close"
    ).onclick=function(){


        panel.remove();


        panel=null;


    };






    document.getElementById(
        "rvn-min"
    ).onchange=function(){


        CONFIG.minDb =
        Number(
            this.value
        );


        checkRange();


    };






    document.getElementById(
        "rvn-max"
    ).onchange=function(){


        CONFIG.maxDb =
        Number(
            this.value
        );


        checkRange();


    };






    document.getElementById(
        "rvn-window"
    ).onchange=function(){


        CONFIG.windowTime =
        Math.max(
            1,
            Number(
                this.value
            )
        );


    };



}





// =====================
// Range Check
// =====================


function checkRange(){


    if(
        CONFIG.minDb >
        CONFIG.maxDb
    ){


        let temp =
        CONFIG.minDb;


        CONFIG.minDb =
        CONFIG.maxDb;


        CONFIG.maxDb =
        temp;



        document.getElementById(
            "rvn-min"
        ).value =
        CONFIG.minDb;



        document.getElementById(
            "rvn-max"
        ).value =
        CONFIG.maxDb;



        console.warn(
            "[RVN] range swapped"
        );


    }



}





// =====================
// UI Update
// =====================


window.RVN_UpdateUI =
function(){


    if(
        !panel
    ){

        return;

    }



    dbText.textContent =
    currentDb.toFixed(1)
    +" dB";



    if(
        video
    ){

        volumeText.textContent =
        Math.round(
            video.volume*100
        )
        +"%";

    }



    if(
        !video
    ){

        statusText.textContent =
        "没有视频";


    }
    else if(
        connected
    ){

        statusText.textContent =
        "运行中";


    }


};





// =====================
// Init
// =====================


function init(){


    console.log(
        "[RVN] init"
    );



    createStyle();


    createUI();


    scanVideo();


}





if(
    document.readyState ===
    "loading"
){

    document.addEventListener(
        "DOMContentLoaded",
        init,
        {
            once:true
        }
    );


}
else{


    init();


}





window.addEventListener(
    "error",
    function(e){


        console.error(
            "[RVN ERROR]",
            e.error ||
            e.message
        );


    }
);



console.log(
    "[RVN] ready"
);



})();
