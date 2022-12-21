<script>

    import { onMount } from "svelte";
    import Header from "$components/Header.svelte"
    import { Swiper, SwiperSlide } from 'swiper/svelte';
    import ReadMoreHeader from '$components/ReadMoreHeader.svelte';
    import SwiperCore, {
        Controller,Mousewheel,Pagination, Keyboard, A11y, Parallax
    } from 'swiper/core';
    import { getContext } from "svelte";
    import 'swiper/css';
    import 'swiper/css/parallax';

    import { fade, slide, blur, fly } from 'svelte/transition';
    import { flip } from 'svelte/animate';
    import chartRank from "$svg/chart_rank.svg";
    import flowChart from "$svg/flow.svg";
    import nytRank from "$svg/nyt_rank.svg";
    import goodreads from "$svg/goodreads.svg";

    import rankingData from "$data/data.csv";
    

	import ButtonSet from "$components/helpers/ButtonSet.svelte";
    let buttonValue;

    SwiperCore.use([Controller, Mousewheel,Pagination,Controller, Keyboard, A11y, Parallax]);
	let sliderEl; // component binding
    let movingBackwards = false;
    let w = false;
    let viewportHeight = null;
	const copy = getContext("copy");
    let onProgress;
    let onInit;
    let popFirst = false;
    let shranked = false;
    let swiper;
    let filtersHidden = false;
    let transform;
    let bookSmall = false;
    let changedSlideEnd
    let activeIndex = 0;
    let height;
    let bookHeight = 200;
    let readMoreVisible = false;
    let doubleTap;
    let slideLength;
    let baseValue;
    let filterUpdated = 0;
    let changeDirection;
    let changeDirectionForwards;
    
    function slideNext(){
        swiper.slideNext();
    }

    function slidePrevious(){
        swiper.slidePrev();
    }

    function handleRankChange(event) {
        let activeSlide
        filterUpdated = filterUpdated + 1;
        swiper.slideTo(baseValue + event.detail.text - 1, 500)
    }

    function handleFilterChange(event) {
        filterUpdated = filterUpdated + 1;
        swiper.slideTo(baseValue, 500)
    }

    let decadeOptions = [
        {"label":"&rsquo;80s","value":"198"},
        {"label":"&rsquo;90s","value":"199"},
        {"label":"&rsquo;00s","value":"200"}
    ]

    let genreOptions = [
        {"label":"Fiction","value":"fiction"},
        {"label":"Non-fiction","value":"nonfiction"}
    ]

    console.log(rankingData[0]["awards"].split(/\r?\n/));

    let countOptions = [1,2,3,4,5,6,7,8,9,10].map(d => {
        return {"value":d};
    });

    let todos = [{ id: "slide-gap"},{id:"rank"},{id:"book"},{id:"read-more"},{id:"title"}];

    let offset = {gap: bookHeight, book:0};
    let topTenActive = false;
    let rankingValue = 1;
    let decadeValue = "198";
    let genreValue = "fiction";
    let dataRankings = [];
    let bookWidth = false;

    let parallaxValue = "50%";

    let readMoreExampleTransform;

    let textTransform = 50;

    $: console.log("clientwidth",bookWidth/1.4)

    $: if(w < 700) {
        parallaxValue = 0//w/2 - (bookWidth/1.4/5) - 70;
    }
    else if(w < 700) {
        parallaxValue = w/2 - (bookWidth/1.4/5) - 50;
    } else {
        parallaxValue = w/2 - (bookWidth/1.4/5);
    }

    $: dataRankings = rankingData.filter(d => {
        return d.decade == decadeValue && d["type"] == genreValue
    }).slice(0,10);

    $: testTransform = transform * (slideLength - 1) % 1;

    $: transformHide = activeIndex
    //* (slideLength - 1) % 1
    $: if(transform * (slideLength - 1) > 5.999){
        readMoreExampleTransform = 1;
    }
    else if(transform * (slideLength - 1) < 4.999999) {
        readMoreExampleTransform = 0;
    }
    else {
        readMoreExampleTransform = transform * (slideLength - 1) % 1
    }

    $: if(activeIndex > copy.intro.length - 1) {
        topTenActive = true;
    }
    else {
        topTenActive = false;
    }

    $: if(activeIndex == 0) {
        popFirst = true;
    } else {
        popFirst = false;
    }

    $: if(activeIndex < (copy.intro.length + copy.ranking.length)){
        filtersHidden = true;
    } else {
        filtersHidden = false;
        baseValue = copy.intro.length + copy.ranking.length;
        let delta = activeIndex - baseValue;
        let slideInRanking = delta % 10;
         rankingValue = slideInRanking + 1;
        // let slideInRanking = activeIndex - copy.intro.length;
        // rankingValue = 10//+copy["ranking"][slideInRanking]["rank"].replace(/\D/g,'');
    }

    // $: if(activeIndex < (copy.intro.length + copy.ranking.length)) {
    //     console.log("hiiii")
    //     let baseValue = copy.intro.length + copy.ranking.length;
    //     let delta = activeIndex - baseValue;
    //     let slideInRanking = delta % 10;
    //     console.log(slideInRanking)
    //     // let slideInRanking = activeIndex - copy.intro.length;
    //     rankingValue = 10//+copy["ranking"][slideInRanking]["rank"].replace(/\D/g,'');
    // }

    $: offset = setBookHeight(transform,activeIndex);
    $: todos = readMoreVisible ? todos.filter(d => d.id == "book" || d.id == "read-more") :[{ id: "slide-gap"},{id:"rank"},{id:"book"},{id:"title"},{id:"read-more"}];

    

    function setBookHeight(trans, active){
        let bookOffset = 100*trans*-1;
        let gap = bookHeight + bookOffset;
        return {gap: gap, book:bookOffset};
    }

    const readMore = () => {
        console.log("reading more")
        readMoreVisible = !readMoreVisible;
        //filtersHidden = !filtersHidden;
        bookSmall = !bookSmall;
        shranked = !shranked
        swiper.allowTouchMove = !swiper.allowTouchMove;
	};

    onInit = (e) => {
        [swiper] = e.detail;
        console.log(swiper);
        swiper.keyboard.enable()
        slideLength = e.detail[0].slides.length
    }


	onMount(async () => {
        
        if(sliderEl){
            console.log(slideEl)
        }

        onProgress = (e) => {
            const [swiper, progress] = e.detail;
            transform = progress;
        }

        doubleTap = (e) => {
            // const [swiper, progress] = e.detail;
            // transform = progress * (copy.intro.length + 5 - 1);
        }

        changeDirection = (e) => {
            movingBackwards = true;
            console.log("changing", e)
        }

        changeDirectionForwards = (e) => {
            movingBackwards = false;
            console.log("changing", e)
        }

        

        changedSlideEnd = (e) => {
            const [swiper] = e.detail;

            activeIndex = swiper.activeIndex;
            if(!filtersHidden) {
                baseValue = copy.intro.length + copy.ranking.length;
                let delta = activeIndex - baseValue;
                let slideInRanking = delta % 10;
                //console.log(slideInRanking)
                rankingValue = slideInRanking;
            }
            if(readMoreVisible) {
                readMore();
            }
        }


 
	});

</script>
<div class="main-swiper {movingBackwards ? "moving-backwards" : ''} {activeIndex == 0 ? 'opening-color' : ''}" bind:clientHeight={height}>
    <div class="header-wrapper {height}" style="transform: translate(0,{height > 1000 ? (activeIndex < copy.intro.length ? 0 : "-100%") : (activeIndex < 1 ? 0 : "-100%")})">
        <Header />
    </div>
    <button class="slide-buttons slide-forward" on:click={slideNext}>
        <div class="carrot">
            <svg viewBox="0 0 16 27" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M2 25L13 13.5L2 2" stroke="#7E7E7E" stroke-width="2.5"/>
            </svg>
        </div>   
    </button>
    <button class="slide-buttons slide-backward" on:click={slidePrevious}>
        <div class="carrot">
            <svg viewBox="0 0 16 27" fill="none" xmlns="http://www.w3.org/2000/svg">
                <path d="M2 25L13 13.5L2 2" stroke="#7E7E7E" stroke-width="2.5"/>
            </svg>
        </div>   
    </button>

    

    <div class="header-wrapper filters" class:filtersHidden>
        <div class="filter">
            <ButtonSet on:message={handleFilterChange} legend={"Decade"} legendPosition={"left"} options={decadeOptions} bind:value={decadeValue} />
        </div>
        <div class="filter">
            <ButtonSet on:message={handleFilterChange} legend={"Genre"} legendPosition={"left"} options={genreOptions} bind:value={genreValue} />
        </div>
        <div class="ranking">
            <ButtonSet on:message={handleRankChange} legend={""} ranking={true} legendPosition={"left"} options={countOptions} bind:value={rankingValue} />
        </div>
    </div>

    <!-- <div class="header-wrapper read-more-footer" class:readMoreVisible on:click={readMore}>
        <p>hello</p>
    </div> -->
    <!-- <div class="book" style="transform: translate(0,{offset.book}px)">
    </div> -->
    <Swiper parallax={true} watchSlidesProgress={true} on:slideNextTransitionStart={changeDirectionForwards} on:slidePrevTransitionStart={changeDirection} on:swiper={onInit} on:progress={onProgress} on:slideChange={changedSlideEnd} on:doubleTap={doubleTap} initialSlide="0"
    >
        {#each copy.intro as card, index}
            <SwiperSlide>
                {@const firstSlide = (index == 0)}
                {@const secondSlide = (index == 1)}

                <div class="exposition-slide {card.center ? "flex-center": "flex-end"} {index == 0 ? 'first-slide' : ''}">
                    <div bind:clientHeight={card["height"]} class="slide-content">
                        {#each card.text as text, index}
                            <p class="{firstSlide ? 'center' : ''}">{@html text.value}</p>
                        {/each}
                        {#if index == 0}
                            <p class="tap">Swipe Right
                                <span class="right-arrow">
                                    <svg viewBox="0 0 22 12" fill="none" xmlns="http://www.w3.org/2000/svg">
                                        <path d="M21.5303 6.53033C21.8232 6.23743 21.8232 5.76256 21.5303 5.46967L16.7574 0.696698C16.4645 0.403804 15.9896 0.403805 15.6967 0.696698C15.4038 0.989591 15.4038 1.46446 15.6967 1.75736L19.9393 6L15.6967 10.2426C15.4038 10.5355 15.4038 11.0104 15.6967 11.3033C15.9896 11.5962 16.4645 11.5962 16.7574 11.3033L21.5303 6.53033ZM6.55671e-08 6.75L21 6.75L21 5.25L-6.55671e-08 5.25L6.55671e-08 6.75Z" fill="black"/>
                                    </svg>                                        
                                </span>

                            </p>
                        {/if}
                    </div>
                    {#if card.img}
                        <div class="img-random" class:popFirst="{popFirst && secondSlide}">
                            <img src="assets/{card.img}" alt="">
                        </div>
                    {/if}
                </div>
                
            </SwiperSlide>
        {/each}

        {#each copy["ranking"] as book, index}
            {@const isBook = (book["book"] === 'true')}
            {@const addReadMore = (book["readMore"] === 'true')}
            {@const rank = book['rank']}

            <SwiperSlide let:data="{{ isActive }}">
                <div class="swiper-slide-flex">

                
                {#if isBook}
                    {#if book["textLocation"] == "above"}
                        <div class="book-rank-text">
                            {#each book.text as text, index}
                                <p>{@html text.value}</p>
                            {/each}
                        </div>
                    {/if}
                    <div class="book-ranked {index == 0 ? "first-book" : ''}" class:shranked>

                        {#each todos as todo (todo.id)}
                            {@const t = {
                                duration: 500,
                                delay: 0
                            }}

                            <div data-swiper-parallax={todo.id == "book" ? parallaxValue : 'null'} class="{todo.id} {readMoreVisible && todo.id == "read-more" ? "readMoreVisible" : ''}" >
                                {#if todo.id == "rank"}
                                    {#if +rank !== 1}
                                        <p class="ranking">
                                            #{rank}
                                        </p> 
                                    {/if}
                                {/if}
                                {#if todo.id == "book"}
                                    <div bind:clientWidth={w} bind:clientHeight={bookWidth} class="book-outer">
                                        <div style="background-image:url(assets/{book.img});" class="book-inner">
                                            <!-- {#if book.img}
                                                <img src="assets/{book.img}" alt="">
                                            {:else}
                                                <img src="assets/things.png" alt="">
                                            {/if} -->
                                        </div>
                                    </div>

                                {/if}

                                {#if todo.id == "title"}
                                    <p class="title-text">{@html book.title}</p>
                                    <p class="author">{book.author}</p>
                                    <p class="count">appears in {book.count} syllabi</p>
                                {/if}

                                {#if todo.id == "read-more"}
                                    <button class="read-more-button" on:click={readMore}>
                                        <ReadMoreHeader />
                                    </button>
                                    {#if addReadMore}
                                        <div class="read-more-content">
                                            <ul>
                                            {#each book.readMoreText as text, index}
                                                {@html text.value}
                                            {/each}
                                            </ul>
                                        </div>
                                    {/if}
                                {/if}
                            </div>
                        {/each}


                    </div>

                {:else}
                    <div class="exposition-slide {book.center ? "flex-center": "flex-end"} {book.id}">
                        <div class="slide-gap">
                        </div>
                        <div class="slide-content">
                            {#each book.text as text, index}
                                <p>{@html text.value}</p>
                            {/each}
                            {#if book.rank == "2a"}
                                <div class="chart-rank-wrapper">
                                    {@html chartRank}
                                </div>
                            {/if}
                            {#if book.rank == "2c"}
                                <div class="chart-rank-wrapper flow-chart">
                                    {@html flowChart}
                                </div>
                                <p>Let&rsquo;s continue on with the rest of the top 10.</p>
                            {/if}

                            {#if book.rank == "7a"}
                                <div class="chart-rank-scroll-wrapper">
                                    <div class="chart-rank-wrapper chart-rank-wrapper-full chart-rank-wrapper-nofull">
                                        {@html goodreads}
                                    </div>
                                </div>
                            {/if}


                            
                            {#if book.rank == "4a"}
                                <div class="chart-rank-scroll-wrapper">
                                    <div class="chart-rank-wrapper chart-rank-wrapper-full">
                                        {@html nytRank}
                                    </div>
                                </div>
                            {/if}
                        </div>
                        {#if book.rank == "1b"}
                            <div class="read-more read-more-example" style="transform:translate(0,{readMoreExampleTransform*100}%);">
                                <div class="read-more-button">
                                    <ReadMoreHeader />
                                </div>
                                <div class="read-more-content">
                                    {#each book.readMoreText as text, index}
                                        <p>{@html text.value}</p>
                                    {/each}
                                </div>
                            </div>
                        {/if}
                    </div>  

                {/if}
                    <!-- <h1>{isActive},{book}</h1> -->
                </div>
            </SwiperSlide>
        {/each}


        {#each copy["explore"] as id, index}
            {#each dataRankings as rankedItem, index}
                <SwiperSlide let:data="{{ isActive }}">
                    {#key filterUpdated}
                        <div class="swiper-slide-flex explore-slide">
                        <div 
                            class="book-ranked" class:shranked>


                            <div data-swiper-parallax="" bind:clientWidth={w} bind:clientHeight={bookWidth} class="book-outer">
                                    <div 
                                        out:fade={{duration: 100}}
                                        in:fly={{duration:500, x:-50, delay:250}}
                                        style="background-image:url(assets/things.png);" class="book-inner"
                                    >
                                    </div>
                            </div>

                            <div class="title">
                                <p
                                    out:fade={{duration: 250}}
                                    in:fly={{duration:500, x:-50, delay:500}}
                                    class="title-text"><i>{rankedItem.title}</i> ({rankedItem.year_y})
                                </p>
                                <p 
                                    out:fade={{duration: 250}}
                                    in:fly={{duration:500, x:-50, delay:750}}

        
                                    class="author">{rankedItem.author_forenames} {rankedItem.author_keyname}
                                </p>
                                <p
                                    out:fade={{duration: 250}}
                                    in:fly={{duration:500, x:-50, delay:1000}}
                                    class="count">appears in 405 syllabi
                                </p>
                            </div>

                            <div class="read-more {readMoreVisible ? "readMoreVisible" : ''}">
                                <button class="read-more-button" on:click={readMore}>
                                    <ReadMoreHeader />
                                </button>
                                <div class="read-more-content">
                                    {#if rankedItem["awards"].length > 0}
                                        <ul>
                                        {#each rankedItem["awards"].split(/\r?\n/) as award}
                                            <li>{award}</li>
                                        {/each}
                                        </ul>
                                    {/if}

                                    

                                </div>
                            </div>
                        </div>
                        </div>
                    {/key}
                </SwiperSlide>
            {/each}
        {/each}


        

    </Swiper>
    
</div>

<style>
    
    .main-swiper .swiper {
    }

    .filtersHidden {
        transform: translate(0,-100%);
    }

    .chart-rank-wrapper {
        margin-left: auto;
        margin-right: auto;
        width: 100%;
        max-width: 350px;
        margin-top: 40px;
    }

    .chart-rank-scroll-wrapper {
        position: relative;
    }

    .chart-rank-wrapper-full {
        max-width: 400px;
        max-height: 300px;
        overflow: scroll;
        position: relative;
        padding-bottom: 100px;
    }

    .chart-rank-wrapper-nofull {
        max-width: 300px;
        width: 100%;
        max-height: 350px;
    }

    .chart-rank-scroll-wrapper:after {
        content: '';
        background: linear-gradient(180deg, rgba(255,252,227,0) 0%, #fffce3 75%);
        position: absolute;
        bottom: 0;
        left: 0;
        right: 0;
        width: 100%;
        height: 100px;
        z-index: 100;
        pointer-events: none;
    }

    .chart-rank-scroll-wrapper:before {
        content: '▼ Scroll for More';
        position: absolute;
        bottom: 10px;
        left: 0;
        right: 0;
        width: 100%;
        z-index: 1000;
        text-align: center;
        font-family: 'Inter';
        font-weight: 600;
        font-size: 12px;
        text-transform: uppercase;
        pointer-events: none;
    }

    

    .book-rank-text {
        max-width: 500px;
        margin: 0 auto;
        margin-top: 100px;
        margin-bottom: 0;
        width: calc(100% - 30px);
    }

    .ranking {
        display: flex;
        width: 100%;
        justify-content: space-around;
        margin-top: 10px;
    }

    .read-more {
        width: calc(100% - 50px);
        margin: 0 auto;
        display: flex;
        flex-direction: column;
        border: 1px solid rgba(204,204,204,.43);
        background: #FEFCEE;
        width: 100%;
        max-width: 500px;
        border-top-left-radius: 10px;
        border-top-right-radius: 10px;
        margin: 0 auto;
        box-shadow: 0px 0px 15px 11px rgba(0,0,0,.03);
        position: absolute;
        top: 100%;
        transform: translate(0, -50px);
        left: 0;
        right: 0;
        min-height:50vh;
        transition: transform .2s;
    }

    .read-more-example {
        top: 0;
        flex-grow: 1;
        position: relative;
        transform: translate(0,0);
        margin-top: 10px;
        min-height: auto;
        max-height: 250px;
        margin-top: auto;
    }

    

    .slide-buttons {
        position: fixed;
        top: 50%;
        z-index: 1000000;
        cursor: pointer;
        transform: translate(0,-50%);
        left: 0;
        right: auto;
        height: 75px;
        width: 50px;
        border: 1px solid rgba(204,204,204,.43);
        background: #FEFCEE;
        border-top-right-radius: 10px;
        border-bottom-right-radius: 10px;
        box-shadow: 0px 0px 15px 11px rgba(0,0,0,.03);
        display: flex;
        flex-direction: column;
        justify-content: center;
    }

    .slide-buttons:hover {
        background: white;
    }

    .opening-color .slide-backward {
        display: none;
    }

    .slide-forward {
        border-top-left-radius: 10px;
        border-bottom-left-radius: 10px;
        border-top-right-radius: 0px;
        border-bottom-right-radius: 0px;
        
        left: auto;
        right: 0;
    }

    .opening-color .slide-forward {
        background: white;
    }

    .slide-buttons .carrot {
        width: 21px;
        margin-left: 8px;
        align-self: center;
    }
    .slide-buttons .carrot svg {
        width: 100%;
    }
    .slide-backward .carrot {
        margin-left: 0;
        margin-right: 8px;
    }
    .slide-backward .carrot svg {
        transform: rotate(180deg);
    }

    

    .slide-buttons .carrot svg path {
        stroke-width: 2px;
    }

    .read-more-button {
        background: none;
        font-family: 'Inter';
        color: #797130;
        letter-spacing: 1px;
        font-size: 14px;
        -webkit-font-smoothing: auto;
        font-weight: 500;
        text-transform: uppercase;
        display: flex;
        justify-content: center;
        padding: 1rem 2rem;
    }

    .exposition-slide {
        height: 100%;
        margin: 0 auto;
        display: flex;
        flex-direction: column;
        justify-content: flex-start;
    }

    .exposition-slide .slide-content {
        max-width: 500px;
        width: calc(100% - 30px);
        margin: 0 auto;
        display: flex;
        flex-direction: column;
        justify-content: flex-end;
        flex-grow: 1;
        padding-bottom: 25px;
        padding-top: 0;
    }



    

    .slide-content p, .book-rank-text p {
        margin-top: 1.5em;
        margin-left: 0;
    }

    .slide-content p:first-of-type, .book-rank-text p:first-of-type {
        margin-top: 0;
    }


    .tap {
        margin: 0 auto;
        width: 100%;
        margin-top: 2rem;
        text-align: center;
    }

    .first-book .read-more {
        display: none;
    }


    .book-inner {
        /* width: auto; */
        /* margin: 0 auto; */
        transition: height .3s;
        /* height: 100%; */
        max-height: 500px;
        height: 100%;
        position: relative;
        /* background: url(assets/god.png); */
        background-size: contain;
        background-repeat: no-repeat;
        background-position: center;
    }

    .opening-color {
        background: #ffefe3;
        background: white;
    }

    .book-inner img {
        margin: 0 auto;
        width: 100%;
        height: 100%;
        transform: rotate(0deg);
    }

    .book-ranked .book {
        height: 1px;
        flex-grow: 1;
        max-height: 500px;
    }

    .book-ranked .book-outer {
        position: relative;
        margin: 0 auto;
        height: 100%;
    }

    .right-arrow {
        display: inline-block;
        width: 21px;
        margin-left: 10px;
    }

    .book-ranked {
        display: flex;
        flex-direction: column;
        flex-grow: 1;
        height: 1px;
        justify-content: center;
    }

    .book-ranked .slide-gap {
        display: none;
    }

    .header-wrapper {
        position: absolute;
        z-index: 10000;
        top: 0;
        left: 0;
        right: 0;
        margin: 0 auto;
        transition: transform .5s .2s;
    }

    .filters {
        display: flex;
        flex-wrap: wrap;
        justify-content: space-around;
        padding-top: 20px;
        max-width: 500px;
        padding-bottom: 50px;
    }

    .read-more-footer {
        height: 100%;
        width: 100%;
        top: 100px;
        background-color: white;
        transform: translate(0,100%);
    }

    .title {
        text-align: center;
        margin-top: 10px;
        margin-bottom: 60px;
    }

    .shranked .book-inner {
        height: 33vh;
    }

    .title-text {
        font-size: 18px;
        -webkit-font-smoothing: auto;
    }

    .author {
        font-size: 18px;
        font-style: normal;
    }

    .count {
        font-size: 12px;
        font-weight: 600;
        color: rgba(0,0,0,.3);
        font-style: normal;
        margin-top: 10px;
        font-family: 'Inter';
    }

    .chart-title {
        font-size: 20px;
    }
    .img-random {
        max-height: 400px;
        height: 1px;
        flex-grow: 1;
        transform: translate(0,50px) rotate(0deg);
        transition: transform .5s;
        margin-top: auto;
    }

    .img-random img{
        height: 100%;
        width: auto;
        margin: 0 auto;
    }

    .swiper-slide-flex {
        display: flex;
        flex-direction: column;
        height: 100%;
        justify-content: center;
    }

    .read-more-content {
        margin-top: 0px;
    }

    .read-more-content ul {
        margin-top: 0px;
    }

    .ranking {
        font-size: 24px;
        color: #999889;
        font-family: 'Inter';
    }

    .flex-center {
        justify-content: center;
    }


    .flex-center .slide-content {
        justify-content: center;
    }
    .explore-slide .book-outer {
        display: flex;
        flex-direction: column;
        margin: 0;
    }

    .explore-slide .book-outer .book-inner {
        margin-top: 100px;
        max-height: none;

    }
    
    .first-book .book-inner {
        height: calc(100% - 50px);
        margin-top: 30px;
    }

    .flow-chart {
        max-width: 240px;
        display: none;
    }

    @media only screen and (max-width: 600px) {

        .book-rank-text {
            margin-top: 0;
        }
		.slide-buttons {
			display: none;
		}

        .read-more-button {
            padding: .5rem 0rem;

        }
        .read-more {
            width: calc(100% - 30px);
            transform: translate(0,-30px);
        }

        .read-more-example {
            max-height: 300px;
        }
	}

    .readMoreVisible {
        transform: translate(0, -100%);
    }

    



</style>