https://www.zhihu.com/question/20466128

public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable

    public Iterator<E> iterator() {
        return new Itr();
    }

      private class Itr implements Iterator<E> {
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor != size;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

            private static class TransformingRandomAccessList<F, T> extends AbstractList<T> implements RandomAccess, Serializable {


        public Iterator<T> iterator() {
            return this.listIterator();
        }

        public ListIterator<T> listIterator(int index) {
            return new TransformedListIterator<F, T>(this.fromList.listIterator(index)) {
                T transform(F from) {
                    return TransformingRandomAccessList.this.function.apply(from);
                }
            };
        }

                    this.function = (Function)Preconditions.checkNotNull(function);

                    https://blog.csdn.net/young4dream/article/details/76794659