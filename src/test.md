```
import React from 'react';
import { configure, mount } from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';
import { setModel, useModel } from './index';

configure({ adapter: new Adapter() });

beforeAll(() => {
  jest.spyOn(console, 'error').mockImplementation((...args) => {
    if (args[0].includes('Warning: An update to %s inside a test was not wrapped in act')) return;
    console.error(...args);
  });
});

test('setModel', () => {
  // 当直接调用时候抛异常
  expect(() => {
    setModel();
  }).toThrow();

  const model = { state: {}, actions: () => ({}) };
  setModel('exist', model);
  setModel('exist', model);
  process.env.NODE_ENV = 'production';
  setModel('exist', model);
  process.env.NODE_ENV = 'test';

  expect(() => {
    setModel('noModel');
  }).toThrow();

  expect(() => {
    setModel('noModelKeys', {});
  }).toThrow();

  expect(() => {
    setModel('noModelActions', { state: {} });
  }).toThrow();
});

test('useModel', () => {
  expect(() => {
    useModel();
  }).toThrow();

  expect(() => {
    useModel('someModel', null);
  }).toThrow();

  expect(() => {
    useModel('modelNotExist');
  }).toThrow();
});

test('component', (done) => {
  // "production" start
  process.env.NODE_ENV = 'production';
  const model = {
    state: {
      count: 0,
    },
    actions: ({ model, setState }) => ({
      showError() {
        setState();
      },
      increase() {
        const { count } = model();
        setState({ count: count + 1 });
      },
      async increaseAsync() {
        const { increase } = model();
        await new Promise((resolve) => setTimeout(resolve, 1000));
        increase();
      },
    }),
  };
  setModel('counter', model);
  function Counter() {
    const { count } = useModel('counter');
    const { showError, increase, increaseAsync } = useModel('counter', true);
    return (
      <>
        <p>{count}</p>
        <button className="show-error" onClick={showError}>
          show error
        </button>
        <button className="increase" onClick={increase}>
          +1
        </button>
        <button className="increase-async" onClick={increaseAsync}>
          +1 async{increaseAsync.loading && '...'}
        </button>
      </>
    );
  }
  const wrapper = mount(<Counter />);
  wrapper.find('.show-error').simulate('click');
  process.env.NODE_ENV = 'test';
  // "production" end

  expect(() => {
    wrapper.find('.show-error').simulate('click');
  }).toThrow();
  wrapper.find('.increase').simulate('click');
  wrapper.find('.increase-async').simulate('click');
  setTimeout(() => {
    wrapper.unmount();
    done();
  }, 1000);
});

```