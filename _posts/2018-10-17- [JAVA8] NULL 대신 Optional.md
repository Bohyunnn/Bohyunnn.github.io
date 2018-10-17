### Null 대신 optional

##### 1. 값이 없는 상황을 처리 하는 방법  
- null값 체크해서 NullPointerException  

```  
public class Main {
	 
	public static void main(String[] args) {
		
	Person bohyun = new Person(); 
	System.out.println(getCar(bohyun));  // return null;
	System.out.println(getCarInsuranceName1(bohyun)); // return "Unknown";
	
	System.out.println(bohyun.getCar().getInsurance().getName()); // [Exception] java.lang.NullPointerException
		
	}
	
	public static Car getCar(Person person){
		return person.getCar();
	}
	
	/**
	 *  Person의 Car이 비어있다면
	 *  => NullPointerException 발생
	 */
	public String getCarInsuranceName(Person person) {
		return person.getCar().getInsurance().getName();
	}
	
	/**
	 *  1. null 확인 체크로 NullPointerException을 줄이기  
	 *  => null 확인 코드 때문에 들여쓰기가 많아진다.
	 *     이런 반복 패턴 코드를 '깊은 의심(deep doubt)' 라고 한다.
	 *     (코드 구조가 엉망, 가독성이 떨어짐)
	 */
	public static String getCarInsuranceName1(Person person){
		if(person != null){
			Car car = person.getCar();
			if(car != null){
				Insurance insurance = car.getInsurance();
				if(insurance != null){
					return insurance.getName();
				}		
			}
		}
		return "Unknown";
	}
	/**
	 * 2. 1과 같은 문제점으로 중첩 if문을 없앤다.
	 * => 출구가 너무 많아져서 유지보스에 어려움이 생긴다.
	 */
	public static String getCarInsuranceName2(Person person) {
		if( person == null ) {
			return "Unknown";
		}
		Car car = person.getCar();
		if ( car == null ) {
			return "Unknown";
		}
		Insurance insurance = car.getInsurance();
		if ( insurance == null ) {
			return "Unknown";
		}
		return insurance.getName();
	}

	public static class Person {
		private Car car;
		public Car getCar() {return car;}
	}
	
	public static class Car {
		private Insurance insurance;
		public Insurance getInsurance() {return insurance;}
	}
	
	public static class Insurance {
		private String name;
		public String getName() {return name;}
	}

	
}


```  
  
##### 2. null 때문에 발생하는 문제
 - 에러의 근원이다  
  : NullPointerException은 자바에서 가장 흔히 발생하는 에러다.  
 - 코드를 어지럽힌다.  
  : 때로는 중첩된 null 확인 코드를 추가해야 하므로 null 때문에 가독성이 떨어진다.  
 - 아무 의미가 없다.  
  : null은 아무 의미도 표현하지 않는다.   
   정적 형식 언어에서 값이 없음을 표현하는 방법으로는 적절하지 않다.  
 - 자바 철학에 위배된다.  
  : 자바는 개발자로부터 모든 포인터를 숨겼다. 하지만 한 가지 예외가 있는데 그것이 null 포인터다.  
 - 형식 시스템에 구멍을 만든다.  
  : null은 무형식이며 정보를 포함하고 있지 않으므로 모든 레퍼런스 형식에 null을 할당할 수 있다.  
    이런 식으로 null이 할당되기 시작하면서 시스템의 다른 부분으로 null이 퍼졌을 때 애초에 null이 어떤 의미로 사용되었는지 알 수 없다.  

##### 3. java8에서 제공하는 것
- `java.util.Optional<T>`  
: 선택형 값 개념의 영향을 받아서 java.util.Optional<T> 새로운 클래스를 제공한다.  

-------------------------------------------------------------------------------------------------------------
### optional 클래스 소개  
- Optional: 선택형값을 캠슐화하는 클래스  
 ex) 어떤 사람이 차를 소유하고 있지 않다면 Person 클래스의 car 변수는 null을 가져야한다.  
    but, Optional을 이용해 null을 할당하는 것이 아니라 Optional을 반환할 수 있다. => `Optional<Car> ` 
- 값이 있으면 Optional 클래스는 값을 감싸고, 값이 없으면 Optional.empty 메소드로 Optional을 반환한다.
- null과 Optional.empty() 
 `null`을 참조하면 NullPointerException이 발생하지만,  
 `Optional.empty()`는 Optional의 객체이므로 Car형식일때 Optional<Car>로 바뀐다. => 값이 없을 수 있음을 명시적으로 표현한다.  
   
 ```    
  /**
	 * 사람이 차를 소유했을 수도 없을수도 있으므로
	 * Optional로 정의함
	 */
	public static class Person {
		private Optional<Car> car;
		public Optional<Car> getCar() {return car;}
	}
	/**
	 * 차가 보험에 가입되어 있을 수도 없을 수도 있으므로 
	 * Optional로 정의함
	 */
	public static class Car {
		private Optional<Insurance> insurance;
		public Optional<Insurance> getInsurance() {return insurance;}
	}
	/**
	 * 보험회사에는 반드시 이름이 있음  
	 * (이름이 없다면 논리적으로 문제가 있는것임)
	 */
	public static class Insurance {
		private String name;
		public String getName() {return name;}
	}
 ```  
 
 -> 메서드 시그너처만 보고 `선택형 값`인지 아닌지 파악이 가능하다.  
 
 ---------------------------------------------------------------------------------------------------------------------
 
 ### Optional 적용 패턴  
 1. Optional 객체 만들기  
 - 빈 Optional (Optional.empty)  
 `ex) Optional<Car> optCar = Optional.empty();`  
 - null이 아닌 값으로 Optional 만들기(Optional.of)  
 `ex) Optional<Car> optCar = Optional.of(car);`  
 car이 null이면 NullPointerException 발생한다.  
 Optional을 사용하지 않았다면 car의 프로퍼티에 접근할때 에러 발생한다.  
 - null값으로 Optional 만들기(Optional.ofNullable)    
 `ex) Optional<Car> optCar = Optional.ofNullable(car);`  
 car이 null이면 빈 Optional 객체를 반환한다.  
 
 
 



