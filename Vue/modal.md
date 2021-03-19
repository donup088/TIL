### Modal 사용
- vue 홈페이지에 들어가면 예시를 볼 수 있다.
- slot 사용
    - 컴포넌트의 내용을 바꿔서 사용할 수 있다.
- Modal 사용
```
<Modal v-if="showModal" @close="showModal = false">
    <h3 slot="header">
        경고!
        <i class="fas fa-times closeModalBtn" @click="showModal=false"></i>
    </h3>
    <div slot="body">입력을 해주세요.</div>
    <div slot="footer">copy right</div>
 </Modal>
```
- Modal 컴포넌트
```
<template>
  <transition name="modal">
    <div class="modal-mask">
      <div class="modal-wrapper">
        <div class="modal-container">

          <div class="modal-header">
            <slot name="header">
              default header
            </slot>
          </div>

          <div class="modal-body">
            <slot name="body">
              default body
            </slot>
          </div>

        </div>
      </div>
    </div>
  </transition>
</template>
```