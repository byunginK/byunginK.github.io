---
layout: post
title: "JPA Transactional"
date: 2022-05-29
categories: [JPA]
---

# Transactional

- Class 전역에 설정을 하게 되면 하위 메소드 전부 JPA 트랜잭션에 포함 되게 된다.
- javax와 spring이 제공하는 2가지 어노테이션이 있지만 **spring trasactional**을 사용하는것이 더 좋다. (사용 가능 메소드가 더 많음)

```java
@Service
@Transactional
public class MemberService {

    @Autowired
    private MemberRepository memberRepository;

    public Long join( Member member ){
        validateDuplicateMember(member);
        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember( Member member ) {
        List<Member> findMembers = memberRepository.findByName(member.getName());
        if(!findMembers.isEmpty()){
            throw new IllegalStateException("이미 존재하는 회원");
        }
    }

    public List<Member> findMembers(){
        return memberRepository.findAll();
    }

    public Member findOne(Long memberId){
        return memberRepository.findOne(memberId);
    }
}
```

- 각 메소드에도 설정 가능 하다.
- 또한 조회를 하는 경우 `Transactional (readOnly = true)`를 사용하면 더티체킹을 하지 않고 등등의 성능 이점이 있다.
- 절대로 insert, update에는 사용 X

```java
@Service
public class MemberService {

    @Autowired
    private MemberRepository memberRepository;

    @Transactional
    public Long join( Member member ){
        validateDuplicateMember(member);
        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember( Member member ) {
        List<Member> findMembers = memberRepository.findByName(member.getName());
        if(!findMembers.isEmpty()){
            throw new IllegalStateException("이미 존재하는 회원");
        }
    }

    @Transactional(readOnly = true)
    public List<Member> findMembers(){
        return memberRepository.findAll();
    }

    @Transactional(readOnly = true)
    public Member findOne(Long memberId){
        return memberRepository.findOne(memberId);
    }
}
```

### 아래 서비스는 조회가 더 많으므로 class의 transactional에 readOnly를 설정하고 save에만 일반 transactional 설정

```java
@Service
@Transactional(readOnly = true)
public class MemberService {

    @Autowired
    private MemberRepository memberRepository;

    @Transactional
    public Long join( Member member ){
        validateDuplicateMember(member);
        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember( Member member ) {
        List<Member> findMembers = memberRepository.findByName(member.getName());
        if(!findMembers.isEmpty()){
            throw new IllegalStateException("이미 존재하는 회원");
        }
    }

    public List<Member> findMembers(){
        return memberRepository.findAll();
    }

    public Member findOne(Long memberId){
        return memberRepository.findOne(memberId);
    }
}
```
