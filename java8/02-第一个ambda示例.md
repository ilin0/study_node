#### 第一个示例  内部类的lambda代替(24-34)

```
public class SwingTest {

    public static void main(String[] args) {

        JFrame jFrame = new JFrame("My JFrame");

        JButton jButton = new JButton("My JButton");

        // 匿名内部类， 形式
        jButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("button pressed!");
            }
        });

        JButton jButton2 = new JButton("My JButton2");

        /**
         * 这里 e 的类型就是 ActionListener 编译器的类型推断
         * ，但并不是所有的时候都能推断，如果离上下文比较远是无法感知的
         * -> 后 如果有多行 就用 {} 内实现
         */
        jButton2.addActionListener(e -> System.out.println("button2 pressed!"));

//        jFrame.add(jButton);
        jFrame.add(jButton2);
        jFrame.pack(); // 适应大小
        jFrame.setVisible(true);
        jFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);


        System.out.println("hello world");
    }
}
```