#sapro2 
```
public class A {
	public int x; // attribute of class A

    public class B { // inner class
        int y;

        public B (int y) { // constructor of inner class
            this.y = y; // this denotes the inner object
        }

        public void printThings () {
        // Inner classes defined inside outer class, can access outer classes methods and attributes
            System.out.println("Value of this B is: " + printB(this));
            // (A.this).x === x, as the inner class has access to outer class attributes
            (A.this).x += y
            System.out.println("Changing A's x value to : " + (A.this).x);
        }
    }

    public A (int x) { //class A constructor
        this.x = x;
    }

    public int printB (B m) { // function of class A that acess B attribute
        return m.y
    }

    public B createB (int y) { //function of class B
        return new B(y);
    }
}
```
```
// To run the above code
public class TestInner {

    int attr1 = 7;

    {int attr2 = attr1;} // this is a block, but not a statement!
    // A block is a group of zero or more statements between balanced braces
    // and can be used anywhere a single statement is allowed

    // int attr3 = attr2; // attr2 is out of its scope

    class Inner { // inner class
    }

    public int m () {

        int locvar = 3;

        /* local classes have only access to "effectively final" local variables
         */

        class LocalInner { // classe locale al metodo m

            int q () {
                System.out.println(attr1); // ok, access to a member of the enclosing class
                System.out.println(locvar); // ok, y is "effectively final"
                return 42;
            }

        }
        // locvar++; // locvar is not effectively final anymore
        return new LocalInner().q(); // weirdly enough, this would still be ok if q() were private!

        // what if m() returned an object of *local* type LocalInner!?
        //return new LocalInner(); // must change the return type of m()
    }

    public static void main(String[] args) {

        Inner I;
        // LocalInner J; // LocalInner only visible within the body of m()

        System.out.println(new TestLocal().m());
    }
}
```