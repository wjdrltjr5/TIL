## 생성자에 매개변수가 많다면 빌더를 고려하라
***
빌더패턴
생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더패턴을 선택하는 게 더 낫다. 매개변수 중 다수가 필수가 아니거나 같은 타입이면 특히 더 그렇다. 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 훨씬 간결하고, 자바빈즈보다 훨씬 안전하다.
``` java
public class NutritionFacts{

    private final int servingSize;
    private final int servings;            
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydreate;

    public static class Builder{
        //필수
        private final int servingSize
        private final int servings;
        
        //선택
        private int calories = 0;
        private int fat = 0;
        private sodium = 0;
        private carbohydreate = 0;

        public Builder(int servingSize, int servings){
            this.servingSize = servingSize;
            this.servings = servings;        
        }
        
        public Builder calories(int val){
            calories = val;
            return this;
        }
        
        public Builder fat(int val){
            fat = val;
            return this;
        }

        public Builder sodium(int val){
            sodium = val;
            return this;
        }

        public Builder carbohydreate(int val){
            carbohydreate = val;
            return this;
        }

        public NutritionFacts build(){
            return new NutritionFacts(this);
        }

        private NutritionFacts(Builder builder){
            servingSize = builder.servingSize;
            servings = builder.servings;
            calories = builder.calories;
            fat = builder.fat;
            sodium = builder.sodium;
            carbohydreate = builder.carbohydreate;
        }
    }
}

NutritionFacts cocaCola = new NutritionFacts.Builder(240,0).calories(100).sodium(35).carbohydreate(27).build();

```
