<template>
  <main :id="$style.login_page">
    <article>
      <div :id="$style.ban_login">
        <h1 :id="$style.ban_login_title">Modifiez votre commentaire</h1>
      </div>
      <form action="" method="post" autocomplete="on" :id="$style.form">
        <div :id="$style.form_each">
          <div :id="$style.bottom_form_first">
            <label for="body" :id="$style.label"> Message </label>
            <textarea
              :id="$style.input"
              type="text"
              v-model="body"
              required
              maxlength="250"
            />
          </div>
        </div>
        <p
          :id="$style.counter_letters"
          v-bind:class="[body.length >= 230 ? $style.danger : $style.safe]"
        >
          {{ body.length }} / 250
        </p>
        <p :id="$style.error">{{ error }}</p>
        <div :id="$style.bottom_form">
          <div
            v-if="body.length >= 2"
            :id="$style.bottom_form_button_login"
            @click="modify"
          >
            <p>modifier</p>
          </div>
          <div :id="$style.bottom_form_button_login_block" v-else>
            <p>modifier</p>
          </div>
          <div :id="$style.bottom_form_button_deny">
            <p @click="backward">Annuler</p>
          </div>
        </div>
      </form>
    </article>
  </main>
</template>

<script>
export default {
  name: "CommentUpdate",
  components: {},
  data() {
    return {
      body: "",
      token: "",
      userId: "",
      role: "",
      error: "",
    };
  },
  created() {
    const storage = localStorage.getItem("user");
    const auth = JSON.parse(storage);
    if (auth === null) {
      return this.$router.push({ path: "/" });
    }
    this.token = auth.token;
    this.userId = auth.userId;
    this.role = auth.role;
    fetch("http://localhost:5000/api/comments/" + this.$route.params.com_id, {
      // getting all the previous value for the comment
      method: "GET",
      headers: new Headers({
        "Content-Type": "application/json",
        Authorization: `Bearer ${this.token}`,
      }),
    })
      .then(async (result_) => {
        const res = await result_.json();
        if (!res.error) {
          this.body = res.comment.body;
        } else {
          this.error = res.error;
        }
        if (res.comment.UserId != this.userId && this.role != true) {
          return this.$router.push({ path: "/Home" });
        }
      })
      .catch((error) => {
        console.log(error);
      });
  },
  methods: {
    backward: function () {
      // go back the the post
      this.$router.push({
        name: "PostEach",
        params: { id: this.$route.params.id },
      });
    },
    modify: function () {
      if (this.token === null) {
        return this.$router.push({ path: "/" });
      }

      const infos = {
        body: this.body,
      };
      fetch("http://localhost:5000/api/comments/" + this.$route.params.com_id, {
        // modifying the comment
        method: "PUT",
        headers: new Headers({
          "Content-Type": "application/json",
          Authorization: `Bearer ${this.token}`,
        }),
        body: JSON.stringify(infos),
      })
        .then(async (result_) => {
          const user = await result_.json();
          if (!user.error) {
            this.error = "";
            this.$router.push({
              name: "PostEach", // once it's done, go back to the post
              params: { id: this.$route.params.id },
            });
          } else {
            this.error = user.error;
          }
        })
        .catch((error) => {
          console.log(error);
        });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" module>
$bg-red: #501b1d;
$bg-blue: #557a95;

h1 {
  font-size: 5em;
}

.danger {
  color: red;
}

.safe {
  color: black;
}

#login_page {
  padding-bottom: 2em;
}

#ban_login {
  margin-bottom: 7em;
  &_under {
    font-size: 2em;
  }
}
#input,
textarea {
  width: 75em;
  height: 3em;
  text-align: center;
  &:focus {
    -webkit-transform: scale(1.05);
    -ms-transform: scale(1.05);
    transform: scale(1.05);
    -webkit-box-shadow: 0rem 0.5rem 2rem 0.1rem lighten(black, 60%);
    box-shadow: 0rem 0.5rem 2rem 0.1rem lighten(black, 60%);
  }
}

#counter_letters {
  margin-top: 1em;
  font-weight: bold;
}
textarea {
  min-height: 15em;
}
#form_each {
  -webkit-box-pack: center;
  -ms-flex-pack: center;
  justify-content: center;
  margin-top: 2em;
}
#bottom_form {
  p {
    margin: 0.5em;
  }
  &_button_deny {
    :hover {
      text-decoration: underline;
      cursor: pointer;
    }
  }
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
  -webkit-box-pack: center;
  -ms-flex-pack: center;
  justify-content: center;
  &_button_login {
    color: darken($bg-blue, 30);
    font-weight: bold;
    cursor: pointer;
    width: 8em;
    border-radius: 5px;
    background: lighten($bg-blue, 30);
    height: auto;
    &:hover {
      text-decoration: underline;
      background: lighten($bg-blue, 10);
    }
    &_block {
      color: darken($bg-red, 10);
      font-weight: bold;
      background: lighten($bg-red, 40);
      border-radius: 5px;
      width: 8em;
      cursor: not-allowed;
    }
  }
}
#label {
  display: block;
  margin-bottom: 0.5em;
  &_del {
    &:hover {
      cursor: pointer;
      font-weight: bold;
      text-decoration: underline;
    }
  }
}

#error {
  color: red;
  text-decoration: underline;
  margin-bottom: 1em;
}
// Media queries
@media all and (max-width: 1200px) {
  #input {
    width: 80%;
  }
}

@media all and (max-width: 650px) {
  h1 {
    font-size: 3em;
  }
  #ban_login {
    margin-bottom: 4em;
    &_under {
      font-size: 1.5em;
    }
  }
  #preview img {
    width: 300px;
    height: 300px;
  }
}
</style>