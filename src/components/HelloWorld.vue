<template>
  <div class="hello">
    ​   ​
    <div class="container">
      <div class="row title">
        <h1>Post managment</h1>
      </div>
      <div class="row">
        <input type="text" class="form-control" v-model="new_title" placeholder="Tytuł wpisu">
        ​
        <textarea type="text" class="form-control" v-model="new_content" placeholder="Treść wpisu">
      </textarea>
        ​
        <a class="btn btn-dark" @click="add_post()">Dodaj wpis</a>
      </div>
      ​
      <div class="row">
        <input type="text" class="form-control" v-model="count_posts" placeholder="Liczba wpisow do pokazania">
        <input type="text" class="form-control" v-model="date_posts" placeholder="Od jakiej daty pokazać wpisy? np. 1907">
        <a class="btn btn-dark" @click="show_posts($event)">Pokaż wpisy</a>
      </div>​
    </div>
    ​     ​
    <ul>
      <li v-for="(post, index) in posts" :key="post.id">
        {{post.title}} - {{post.content}}
        <a class="btn btn-dark" :post_id="post.post_id" @click="delete_post($event,index)">Usuń wpis</a>
      </li>
    </ul>
    ​
    ​
    ​
  </div>
</template>
​
<script>
  import db from './firebaseInit'
  export default {
    name: 'HelloWorld',
    data () {
      return {
        postsRef: '',
        usersRef : '',
        count_posts:0,
        new_title : '',
        date_posts : '',
        new_content : '',
        posts : [],
      }
    },
    beforeCreate(){
      console.log('before Created');
    },
    created (){

      // przypisanie zaczepu kolekcji do zmiennej
      this.postsRef = db.collection('posts');
      this.usersRef = db.collection('users');

      // ograniczenie wywołania zapytania do wartości pożądanych przez nas
      // posts.where('klucz_elementu_w_documencie','porównanie','wartośc_el_w_doc')

      // praktycznee zastosowanie
      // users.where('age','>','23')
      // z limitem
      // users.where('date','==','1970').limit(3)

      // wywołanie fukncji onSnapshot przy każdej interakcji z elementami posts w bazie danych
      db.collection('posts').onSnapshot( function(querySnapshot) {
        querySnapshot.forEach( function (doc) {
          console.log(doc.data().title)
        })
      });


      // wywołanie onSnapshot tylko i wyłącznie prrzy dodawaniu
      db.collection('posts').onSnapshot( {includeMetadataChanges: true}, function(snapshot) {
        snapshot.docChanges().forEach( function (change) {

          if (change.type == "added") {
            console.log(change.doc.data().title)
          }
          console.log(change);

        })
      });
    console.log('created');
    // db.collection('posts').get().then( (querySnapshot) => {
      //   querySnapshot.forEach( (doc) => {
      //     console.log(doc.data());
          //     const data = {
      //       'post_id' : doc.data().post_id,
      //       'title' : doc.data().title,
      //       'content' : doc.data().content
      //     }
      //     this.posts.push(data)

      //   })
      // })
    },
    beforeMount (){
      console.log('before mounted');
    },
    mounted (){
      console.log('mounted');
    },
    methods : {
      show_posts(){
        this.posts = []
        console.log(this.postsRef)
        console.log(this.count_posts)
        console.log(this.date_posts)

      this.count_posts = Number(this.count_posts);
        console.log(this.count_posts);

          this.postsRef.where('date','>',this.date_posts).limit(this.count_posts)
            .get()
            .then( (querySnapshot) => {
              querySnapshot.forEach( (doc) => {
                const data = {
                  'post_id' : doc.data().post_id,
                  'title' : doc.data().title,
                  'content' : doc.data().content
                };
                this.posts.push(data);


              })
            })
      },
    delete_post(event,index){
    console.log(index);
    var post_index = index;
    var post_id = event.target.attributes.post_id.value;

      db.collection('posts').get().then( (querySnapshot) => {
        querySnapshot.forEach( (doc) => {

          if (doc.id == post_id) {
            doc.ref.delete();

            // this.posts.splice(post_index,1);

            // wywołanie usunięcia elementu z tablicy po stronie przeglądarki - dedykowane rozwiązanie vue
            this.$delete(this.posts,index);

          }
        })
      });
      },
    add_post(){
    db.collection('posts').add({
      title : this.new_title,
      content : this.new_content
    }).then( docRef => {
        db.collection('posts').doc(docRef.id).set({
          post_id : docRef.id,
          title : this.new_title,
          content : this.new_content,
          date : '2009'
        });
        // console.log(docRef.id)
      const data = {
        post_id : docRef.id,
        title : this.new_title,
        content : this.new_content
      };
      this.posts.push(data);
    }).catch( error => {
      console.log('Coś poszło nie tak z dodawaniem do posts',error);
    });
    // .then(docRef => {
    //   console.log('Udało Ci się dodać wpis -',docRef.id)
    //   // db.collection('posts').doc(docRef.id).set({
    //   //   post_id : docRef.id,
    //   //   title : this.new_title,
    //   //   content : this.new_content
    //   // })

    //   const data = {
    //     title : this.new_title,
    //     content : this.new_content
    //   }
    //   this.posts.push(data);
    // })
    // .catch(error => {
    //   console.log('Wystąpił błąd podczas dodawania wpisu', error)
    // })
  }
  }
  }
</script>
​
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
a.btn-dark{
  color: white;
  margin:5px;
}
a.btn-dark:hover{
  color: green;
}
.form-control{
  margin: 5px;
}
.title{
    text-align: center;
  }
  .title{
    justify-content: center;
  }
</style>
