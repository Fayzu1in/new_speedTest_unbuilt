<template>
  <div>
    <div class="navbar container">
      <div class="navbar__left">
        <img src="@/assets/burger.png" alt="" />
        <img src="@/assets/allplayLogo.png" alt="" />
      </div>
      <div class="navbar__center">
        <ul>
          <li><a href="">ТВ</a></li>
          <li><a href="">Кинотеатр</a></li>
          <li><a href="">GO</a></li>
          <li><a href="">Радио</a></li>
          <li><a href="">Я смотрю</a></li>
          <li><a href="">Search</a></li>
        </ul>
      </div>
      <div class="navbar__right">
        <div class="navbar__right-left">
          <p>RU</p>
          <p>ENG</p>
        </div>
        <div class="navbar__right-right">
          <img src="@/assets/user.png" alt="" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
};
</script>

<style lang="scss" scoped>
.navbar {
  background-color: #000;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-top: 10px;
  padding-bottom: 10px;
  &__left {
    display: flex;
    align-items: center;
    img {
      margin-left: 5px;
      margin-right: 5px;
    }
  }
  &__center {
    ul {
      display: flex;
      list-style-type: none;

      a {
        color: grey;
        text-decoration: none;
        font-size: 18px;
        margin-left: 23px;
        margin-right: 23px;
        transition: 0.3s;
        &:hover {
          color: #fff;
        }
      }
    }
  }
  &__right {
    display: flex;
    align-items: center;

    &-left {
      color: #fff;
      display: flex;
      p {
        margin-right: 5px;
        margin-left: 5px;
        font-size: 14px;
      }
    }
    &-right {
      margin-left: 5px;
      margin-right: 5px;
    }
  }
}
</style>
