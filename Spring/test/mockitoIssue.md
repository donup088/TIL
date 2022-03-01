## spring test에 mockito를 사용하면서 발생하였던 이슈들

### any() 에서 null을 return 해버리면서 nullpointerexception 발생
- 테스트 코드
    ```
    @Test
    @DisplayName("개인정보 수정 성공")
    public void update() {
        given(userRepository.findByIdWithAvatar(any())).willReturn(Optional.of(USER));
        given(schools.getOne(any())).willReturn(MockSchool.School1.SCHOOL);

        userService.update(UPDATE_REQUEST, USER);

        verify(userValidator).updateValidate(any(), any());
        verify(avatarService).update(any(), any(),any());
    }
    ```
- 문제가 되는 코드
    - boolean 값에 any()를 넣을 수 없음.
    ```
    public void update(boolean avatarChanged, MultipartFile updatedAvatar, User findUser) {
        if (avatarChanged) {
            deleteOne(findUser.getAvatar());
            Avatar avatar = upload(updatedAvatar);
            findUser.updateAvatar(avatar);
        }
    }
    ```
- 해결
    - primitive type -> wrapper type
    - boolean 자료형을 null 값도 받을 수 있도록 Boolean 으로 바꿔주었다.