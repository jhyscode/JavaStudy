HashMap��JDK1.8֮ǰ��ʵ�ַ�ʽ ����+����,������JDK1.8���HashMap�����˵ײ��Ż�,��Ϊ���� ����+����+�����ʵ��,��Ҫ��Ŀ������߲���Ч�ʡ�

1.�̳й�ϵ

public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable

2.����&���췽�� //���������޶�ֵ ���ڵ�������8ʱ��תΪ������洢 static final int TREEIFY_THRESHOLD = 8; //���ڵ���С��6ʱ��תΪ���������洢 static final int UNTREEIFY_THRESHOLD = 6; //�������С����Ϊ 64 static final int MIN_TREEIFY_CAPACITY = 64; //HashMap������ʼ��С static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16 //HashMap�������� static final int MAXIMUM_CAPACITY = 1 << 30; //��������Ĭ�ϴ�С static final float DEFAULT_LOAD_FACTOR = 0.75f; //Node��Map.Entry�ӿڵ�ʵ���� //�ڴ˴洢���ݵ�Node����������2���� //ÿһ��Node���ʶ���һ���������� transient Node<K,V>[] table; //HashMap��С,������HashMap����ļ�ֵ�ԵĶ��� transient int size; //HashMap���ı�Ĵ��� transient int modCount; //��һ��HashMap���ݵĴ�С int threshold; //�洢�������ӵĳ��� final float loadFactor;

//Ĭ�ϵĹ��캯�� public HashMap() { this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted } //ָ��������С public HashMap(int initialCapacity) { this(initialCapacity, DEFAULT_LOAD_FACTOR); } //ָ��������С�͸������Ӵ�С public HashMap(int initialCapacity, float loadFactor) { //ָ����������С������С��0,�����׳�IllegalArgumentException�쳣 if (initialCapacity < 0) throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity); //�ж�ָ����������С�Ƿ����HashMap���������� if (initialCapacity > MAXIMUM_CAPACITY) initialCapacity = MAXIMUM_CAPACITY; //ָ���ĸ������Ӳ�����С��0��ΪNull�����ж��������׳�IllegalArgumentException�쳣 if (loadFactor <= 0 || Float.isNaN(loadFactor)) throw new IllegalArgumentException("Illegal load factor: " + loadFactor);

    this.loadFactor = loadFactor;
    // ���á�HashMap��ֵ������HashMap�д洢���ݵ������ﵽthresholdʱ������Ҫ��HashMap�������ӱ���
    this.threshold = tableSizeFor(initialCapacity);
}
//����һ��Map����,��Map������Ԫ��Map.Entryȫ�����ӽ�HashMapʵ����
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    //�˹��췽����Ҫʵ����Map.putAll()
    putMapEntries(m, false);
}
3.Node����������ʵ�� //ʵ����Map.Entry�ӿ� static class Node<K,V> implements Map.Entry<K,V> { final int hash; final K key; V value; Node<K,V> next; //���캯�� Node(int hash, K key, V value, Node<K,V> next) { this.hash = hash; this.key = key; this.value = value; this.next = next; }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
    //equals���ԶԱ�
    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}