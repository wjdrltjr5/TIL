# WebMvcTest시 40X에러

```Java

@WebMvcTest(controllers = ScheduleController.class)
class ScheduleControllerTest {
...
}
```

이런식으로 테스트할때 @WebMvcTest는 컨트롤러 테스트에 필요한 빈만을 가져오기 때문에 Security Config파일을 불러오지 않고 자동으로 구성되는 설정파일들을 사용

다음과 같이 해결

```Java
@WebMvcTest(controllers = ScheduleController.class)
@WithMockUser // mock 유저를 설정하여 401 에러 해결
class ScheduleControllerTest {
    ...

    @Test
    @DisplayName("신규 일정을 생성한다")
    void createSchedule() throws Exception {
        //given
        ScheduleCreateRequest request = ScheduleCreateRequest.builder()
                .title("제목")
                .memo("메모")
                .dueDate(LocalDate.of(2024, 12, 23))
                .state("Ongoing")
                .build();
        //when
        //then
        mockMvc.perform(
                        post("/schedule")
                                .content(objectMapper.writeValueAsString(request))
                                .contentType(MediaType.APPLICATION_JSON)
                                .with(csrf()) // 403 에러 해결
                )
                .andDo(print())
                .andExpect(status().isOk());
    }
    ...
}
```
