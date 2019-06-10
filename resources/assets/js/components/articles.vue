<template>
<div>
    <h2>Articles</h2>

    <form @submit.prevent="addArticle" class="mb-3">
        <div class="form-group">
            <input type="text" class="form-control" placeholder="Title" v-model="article.title">
        </div>

        <div class="form-group">
            <textarea type="text" class="form-control" placeholder="Body" v-model="article.body"></textarea>
        </div>

        <button type="submit" class="btn btn-light btn-block">Save</button>
    </form>

    <nav aria-label="Page navigation example">
        <ul class="pagination">
            <li v-bind:class="[{disabled: !pagination.prev_page_url}]" class="page-item"><a class="page-link" href="#" @click="fetchArticles(pagination.prev_page_url)">Previous</a></li>
            <li class="page-item disabled"><a class="page-link text-dark" href="#">Page {{ pagination.current_page }} of {{ pagination.last_page }}</a></li>
            <li v-bind:class="[{disabled: !pagination.next_page_url}]" class="page-item"><a class="page-link" href="#" @click="fetchArticles(pagination.next_page_url)">Next</a></li>
        </ul>
    </nav>

    <div class="card card-body mb-2" v-for="article in articles" v-bind:key="article.id">
        <!-- <h3>{{ article.title }}</h3>
        <p>{{ article.body }}</p> -->

        
      <!-- <div class="panel-heading">
        <h4 class="panel-title">
          <a data-toggle="collapse" data-parent="#accordion" :href="'#collapse-'+article.id" aria-expanded="true" :aria-controls="'#collapse-'+article.id">{{ article.title }}</a>
        </h4>
      </div>
      <div :id="'collapse-'+article.id" class="collapse show" >
        <div class="panel-body">
          {{ article.body }}
        </div>
      </div> -->
    

        <div class="accordion" id="accordionExample">
  <div class="card">
    <div class="card-header" id="headingOne">
      <h5 class="mb-0">
        <button class="btn btn-link" type="button" data-toggle="collapse" :data-target="'#collapse-'+article.id" aria-expanded="true" aria-controls="collapseOne">
          {{ article.title }}
        </button>
      </h5>
    </div>

    <div :id="'collapse-'+article.id" class="collapse" aria-labelledby="headingOne" data-parent="#accordionExample">
      <div class="card-body">
         {{ article.body }}

         <hr>
        <button @click="editArticle(article)" class=" btn btn-outline-warning mb-2">Edit</button><br>
        <button @click="deleteArticle(article.id)" class=" btn btn-outline-danger">Delete</button>
      </div>
    </div>
  </div>
</div>
        
    </div>
</div>
</template>

<script>
export default {
    data() 
    {
        return{
            articles: [],
            article: {
                id: '',
                title: '',
                body: ''
            },
            article_id: '',
            pagination: {},
            edit: false
        }
    },

    created()
    {
        this.fetchArticles();
    },

    methods: 
    {
        fetchArticles(page_url)
        {
            let vm = this;
            page_url = page_url || '/api/articles'
            fetch(page_url)
            .then(res => res.json())
            .then(res =>
            {
                console.log(res.data);
                this.articles = res.data;
                vm.makePagination(res.meta, res.links);
            })
            .catch(err => console.log(err));
        },

        makePagination(meta, links)
        {
            let pagination = 
            {
                current_page: meta.current_page,
                last_page: meta.last_page,
                next_page_url: links.next,
                prev_page_url: links.prev
            }

            this.pagination = pagination;
        },

        deleteArticle(id)
        {
            if(confirm('Are You Sure?'))
            {
                fetch(`api/article/${id}`,
                {
                    method: 'delete'
                })
                .then(res =>res.json())
                .then(data => {
                    alert('Article Removed');
                    this.fetchArticles();
                })
                .catch(err =>console.log(err));
            }
        },

        addArticle()
        {
            if(this.edit === false)
            {
                //add
                fetch('api/article', 
                {
                    method: 'post',
                    body: JSON.stringify(this.article),
                    headers:
                    {
                        'content-type': 'application/json'
                    }
                })
                .then(res => res.json())
                .then(data => 
                {
                    this.article.title = "";
                    this.article.body = "";
                    alert('Article Added');
                    this.fetchArticles();
                })
                .catch(err => console.log(err)); 
            }
            else
            {
                //update
                fetch('api/article', 
                {
                    method: 'put',
                    body: JSON.stringify(this.article),
                    headers:
                    {
                        'content-type': 'application/json'
                    }
                })
                .then(res => res.json())
                .then(data => 
                {
                    this.article.title = "";
                    this.article.body = "";
                    alert('Article Updated');
                    this.fetchArticles();
                })
                .catch(err => console.log(err)); 
            }
        },

        editArticle(article)
        {
            this.edit = true;
            this.article.id = article.id;
            this.article.article_id = article.id;
            this.article.title = article.title;
            this.article.body = article.body;
        }
    }

}
</script>
