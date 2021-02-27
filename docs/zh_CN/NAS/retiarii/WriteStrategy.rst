自定义 Strategy
========================

要编写新策略，应该继承基本策略类 ``BaseStrategy``，然后实现成员函数 ``run``。 此成员函数将 ``base_model`` 和 ``applied_mutators`` 作为输入参数， 并将用户在 ``applied_mutators`` 中指定的 Mutator 应用到 ``base_model`` 中生成新模型。 当应用一个 Mutator 时，应该与一个 sampler 绑定（例如，``RandomSampler``）。 每个 sampler 都实现了从候选值中选择值的 ``choice`` 函数。 The ``choice`` functions invoked in mutators are executed with the sampler.

Below is a very simple random strategy, which makes the choices completely random.

.. code-block:: python

    from nni.retiarii import Sampler

    class RandomSampler(Sampler):
        def choice(self, candidates, mutator, model, index):
            return random.choice(candidates)

    class RandomStrategy(BaseStrategy):
        def __init__(self):
            self.random_sampler = RandomSampler()

        def run(self, base_model, applied_mutators):
            _logger.info('stargety start...')
            while True:
                avail_resource = query_available_resources()
                if avail_resource > 0:
                    model = base_model
                    _logger.info('apply mutators...')
                    _logger.info('mutators: %s', str(applied_mutators))
                    for mutator in applied_mutators:
                        mutator.bind_sampler(self.random_sampler)
                        model = mutator.apply(model)
                    # 运行模型
                    submit_models(model)
                else:
                    time.sleep(2)

You can find that this strategy does not know the search space beforehand, it passively makes decisions every time ``choice`` is invoked from mutators. If a strategy wants to know the whole search space before making any decision (e.g., TPE, SMAC), it can use ``dry_run`` function provided by ``Mutator`` to obtain the space. An example strategy can be found :githublink:`here <nni/retiarii/strategy/tpe_strategy.py>`.

After generating a new model, the strategy can use our provided APIs (e.g., ``submit_models``, ``is_stopped_exec``) to submit the model and get its reported results. More APIs can be found in `API References <./ApiReference.rst>`__.