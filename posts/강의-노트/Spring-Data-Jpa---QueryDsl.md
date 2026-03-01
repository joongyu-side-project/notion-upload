# Spring Data Jpa + QueryDsl

- 기본적으로 QueryDsl를 사용하는 방법은 사용자 정의 리포지토리를 생성해서 사용하는 것이 일반적이다
  1. 사용자 정의 인터페이스 작성
  1. 사용자 정의 인터페이스 구성
  1. 스프링 데이터 리포지토리에 사용자 정의 인터페이스 상속

```java
public interface MemberRepository extends JpaRepository<Member, Long> , MemberRepositoryCustom {
        // select m from Member m where m.username
        List<Member> findByUsername(String username);
}



--

public interface MemberRepositoryCustom {
    List<MemberTeamDto> search(MemberSearchCondition condition);

}
__

@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberRepositoryCustom {

    private final JPAQueryFactory queryFactory;

    @Override
    public List<MemberTeamDto> search(MemberSearchCondition condition) {
        return queryFactory
            .select(new QMemberTeamDto(
                member.id.as("memberId"),
                member.username.as("username"),
                member.age,
                team.id.as("teamId"),
                team.name.as("teamName")
            ))
            .from(member)
            .leftJoin(member.team, team)
            .where(
                usernameEq(condition.getUserName()),
                teamNameEq(condition.getTeamName()),
                ageGoe(condition.getAgeGoe()),
                ageLoe(condition.getAgeLoe())
            )
            .fetch();
    }

    /// Predicate 보다 BoooleanExpression을 사용하는게 조합 상 좋다.
    private BooleanExpression usernameEq(String userName) {
        return hasText(userName) ? member.username.eq(userName) : null;
    }

    private BooleanExpression teamNameEq(String teamName) {
        return hasText(teamName) ? team.name.eq(teamName) : null;
    }

    private BooleanExpression ageGoe(Integer ageGoe) {
        return ageGoe != null ? member.age.goe(ageGoe) : null;
    }

    private BooleanExpression ageLoe(Integer ageLoe) {
        return ageLoe != null ? member.age.loe(ageLoe) : null;
    }


}
```






