<template>
   <div class="container">
        <div class="card-panel teal lighten-2">
            <h3>Welcome to <b>iHack 4.0</b> Web Player</h3>
        </div>
        <div class="card-panel ">
            <router-link tag="a" to="/vote" title="go back to vote" id="addButton" 
                class="btn-floating btn-large waves-effect waves-light red">
                <i class="material-icons">arrow_back </i>
            </router-link>
            <h5>Add New Song</h5>
            <div class="row">
                <div class="col s12 m6 offset-m3 l4 offset-l4 z-depth-6">
                    <search-bar @newSearch="videoSearch"></search-bar>
                </div>
            </div>
            <ul>
                <video-list-item
                v-for="video in videos"
                :video="video"
                :imgUrl="video.snippet.thumbnails.medium.url"
                :videoTitle="video.snippet.title"
                :key="video.etag"
                @videoSelected="addToThePlaylist"
                ></video-list-item>
            </ul>
            
        </div>
   </div> 
</template>

<script>
    import { YOUTUBE_API_KEY } from '../config';
    import YouTubeSearch from 'youtube-api-search';
    import SearchBar from './SearchBar'
    import VideoListItem from './VideoListItem'
    import {db} from '../db';
    import firebase from 'firebase'

    export default {
    name: 'newsong',
    components: {SearchBar, VideoListItem},
    firebase: {
        items: db.ref('songs')
    },
    data(){
        return {
        videos: [],
        selectedVideo: null,
        }
    },
    created() {
        //this.videoSearch('songs');
    },
    methods:{
        videoSearch(searchTerm) {
            YouTubeSearch( {key: YOUTUBE_API_KEY, term: searchTerm}, (videos) => {
                videos.forEach(video => {
                    video.alreadyInPlaylist = this.isPresentInPlaylist(video)
                })            
                this.videos = videos;
                //console.log(videos)
                // // get the first video (before one is selected --this is the default)
            });
        },
        addToThePlaylist (video) {
            // TODO send to the firestore
            this.videos.splice(this.videos.indexOf(video),1);
            let videoObject = {
                user: firebase.auth().currentUser.uid,
                video,
                upvotes: ['a', firebase.auth().currentUser.uid],
                downvotes: ['a'],
                status: 0,
                name: video.snippet.title,
            }
            // console.log(firebase.auth().currentUser.uid)
            this.$firebaseRefs.items.push(videoObject);
        },
        isPresentInPlaylist(video) {
            return this.items.some(addedSong => this.sameVideo(addedSong.video, video))
        },
        sameVideo(video1, video2) {
            return (video1.id.kind === video2.id.kind && video1.id.videoId === video2.id.videoId)
        }
    }
    }
</script>