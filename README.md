# The RSpec Style Guide

This RSpec style guide outlines the recommended best practices for real-world
programmers to write code that can be maintained by other real-world
programmers.

## How to Read This Guide

The guide is separated into sections based on the different pieces of an entire
spec file. There was an attempt to omit all obvious information, if anything is
unclear, feel free to open an issue asking for further clarity.

## A Living Document

Per the comment above, this guide is a work in progress - some rules are simply
lacking thorough examples, but some things in the RSpec world change week by
week or month by month. With that said, as the standard changes this guide is
meant to be able to change with it.

## Table of Contents

  * [Layout](#layout)
  * [Example Structure](#example-structure)
  * [Example Group Structure](#example-group-structure)
  * [Naming](#naming)
  * [Matchers](#matchers)
  * [Rails](#rails)
      * [Views](#views)
      * [Controllers](#controllers)
      * [Models](#models)
      * [Mailers](#mailers)
  * [Recommendations](#recommendations)

## Layout

  * <a name="empty-lines-after-describe"></a>
    Do not leave empty lines after `feature`, `context` or `describe`
    descriptions. It doesn't make the code more readable and lowers the
    value of logical chunks.
    <sup>[[link](#empty-lines-after-describe)]</sup>

    ```ruby
    # bad
    describe Article do

      describe '#summary' do

        context 'when there is a summary' do

          it 'returns the summary' do
            # ...
          end
        end
      end
    end

    # good
    describe Article do
      describe '#summary' do
        context 'when there is a summary' do
          it 'returns the summary' do
            # ...
          end
        end
      end
    end
    ```

  * <a name="empty-lines-between-describes"></a>
    Leave one empty line between `feature`, `context` or `describe` blocks.
    Do not leave empty line after the last such block in a group.
    <sup>[[link](#empty-lines-between-describes)]</sup>

    ```ruby
    # bad
    describe Article do
      describe '#summary' do
        context 'when there is a summary' do
          # ...
        end
        context 'when there is no summary' do
          # ...
        end

      end
      describe '#comments' do
        # ...
      end
    end

    # good
    describe Article do
      describe '#summary' do
        context 'when there is a summary' do
          # ...
        end

        context 'when there is no summary' do
          # ...
        end
      end

      describe '#comments' do
        # ...
      end
    end
    ```

  * <a name="empty-lines-after-let"></a>
    Leave one empty line after `let`, `subject`, and `before`/`after` blocks.
    <sup>[[link](#empty-lines-after-let)]</sup>

    ```ruby
    # bad
    describe Article do
      subject { FactoryBot.create(:some_article) }
      describe '#summary' do
        # ...
      end
    end

    # good
    describe Article do
      subject { FactoryBot.create(:some_article) }

      describe '#summary' do
        # ...
      end
    end
    ```

  * <a name="let-grouping"></a>
    Only group `let`, `subject` blocks and separate them from `before`/`after`
    blocks. It makes the code much more readable.
    <sup>[[link](#let-grouping)]</sup>

    ```ruby
    # bad
    describe Article do
      subject { FactoryBot.create(:some_article) }
      let(:user) { FactoryBot.create(:user) }
      before do
        # ...
      end
      after do
        # ...
      end
      describe '#summary' do
        # ...
      end
    end

    # good
    describe Article do
      subject { FactoryBot.create(:some_article) }
      let(:user) { FactoryBot.create(:user) }

      before do
        # ...
      end

      after do
        # ...
      end

      describe '#summary' do
        # ...
      end
    end
    ```

  * <a name="empty-lines-around-it"></a>
    Leave one empty line around `it`/`specify` blocks. This helps to separate
    the expectations from their conditional logic (contexts for instance).
    <sup>[[link](#empty-lines-around-it)]</sup>

    ```ruby
    # bad
    describe '#summary' do
      let(:item) { double('something') }

      it 'returns the summary' do
        # ...
      end
      it 'does something else' do
        # ...
      end
      it 'does another thing' do
        # ...
      end
    end

    # good
    describe '#summary' do
      let(:item) { double('something') }

      it 'returns the summary' do
        # ...
      end

      it 'does something else' do
        # ...
      end

      it 'does another thing' do
        # ...
      end
    end
    ```

  * <a name="leading-subject"></a>
    When `subject` is used, it should be the first declaration in the example group.
    <sup>[[link](#leading-subject)]</sup>

    ```ruby
    # bad
    describe Article do
      before do
        # ...
      end

      let(:user) { FactoryBot.create(:user) }
      subject { FactoryBot.create(:some_article) }

      describe '#summary' do
        # ...
      end
    end

    # good
    describe Article do
      subject { FactoryBot.create(:some_article) }
      let(:user) { FactoryBot.create(:user) }

      before do
        # ...
      end

      describe '#summary' do
        # ...
      end
    end
    ```

## Example Group Structure

  * <a name="use-contexts"></a>
    Use contexts to make the tests clear, well organized, and easy to
    read.
    <sup>[[link](#use-contexts)]</sup>

    ```ruby
    # bad
    it 'has 200 status code if logged in' do
      expect(response).to respond_with 200
    end

    it 'has 401 status code if not logged in' do
      expect(response).to respond_with 401
    end

    # good
    context 'when logged in' do
      it { is_expected.to respond_with 200 }
    end

    context 'when logged out' do
      it { is_expected.to respond_with 401 }
    end
    ```

  * <a name="context-cases"></a>
    `context` blocks should pretty much always have an opposite negative case.
    It is a code smell if there is a single context (without a matching
    negative case), and this code needs refactoring, or may have no purpose.
    <sup>[[link](#context-cases)]</sup>

    ```ruby
    # bad - needs refactoring
    describe '#attributes' do
      context 'the returned hash' do
        it 'includes the display name' do
          # ...
        end

        it 'includes the creation time' do
          # ...
        end
      end
    end

    # bad - the negative case needs to be tested, but isn't
    describe '#attributes' do
      context 'when display name is present' do
        before do
          subject.display_name = 'something'
        end

        it 'includes the display name' do
          # ...
        end
      end
    end

    # good
    describe '#attributes' do
      subject { FactoryBot.create(:article) }

      specify do
        expect(subject.attributes).to include subject.display_name
        expect(subject.attributes).to include subject.created_at
      end
    end

    describe '#attributes' do
      context 'when display name is present' do
        before do
          subject.display_name = 'something'
        end

        it 'includes the display name' do
          # ...
        end
      end

      context 'when display name is not present' do
        before do
          subject.display_name = nil
        end

        it 'does not include the display name' do
          # ...
        end
      end
    end
    ```

  * <a name="let-blocks"></a>
    Use `let` and `let!` for data that is used across several examples in an
    example group. Use `let!` to define variables even if they are not
    referenced in some of the examples, e.g. when testing balancing negative
    cases. Do not overuse `let`s for primitive data, find the balance between
    frequency of use and complexity of the definition.
    <sup>[[link](#let-blocks)]</sup>

    ```ruby
    # bad
    it 'finds shortest path' do
      tree = Tree.new(1 => 2, 2 => 3, 2 => 6, 3 => 4, 4 => 5, 5 => 6)
      expect(dijkstra.shortest_path(tree, from: 1, to: 6)).to eq([1, 2, 6])
    end
    
    it 'finds longest path' do
      tree = Tree.new(1 => 2, 2 => 3, 2 => 6, 3 => 4, 4 => 5, 5 => 6)
      expect(dijkstra.longest_path(tree, from: 1, to: 6)).to eq([1, 2, 3, 4, 5, 6])
    end
    
    # good
    let(:tree) { Tree.new(1 => 2, 2 => 3, 2 => 6, 3 => 4, 4 => 5, 5 => 6) }
    
    it 'finds shortest path' do
      expect(dijkstra.shortest_path(tree, from: 1, to: 6)).to eq([1, 2, 6])
    end
    
    it 'finds longest path' do
      expect(dijkstra.longest_path(tree, from: 1, to: 6)).to eq([1, 2, 3, 4, 5, 6])
    end
    ```

  * <a name="instance-variables"></a>
    Use `let` definitions instead of instance variables.
    <sup>[[link](#instance-variables)]</sup>

    ```ruby
    # bad
    before { @name = 'John Wayne' }

    it 'reverses a name' do
      expect(reverser.reverse(@name)).to eq('enyaW nhoJ')
    end

    # good
    let(:name) { 'John Wayne' }

    it 'reverses a name' do
      expect(reverser.reverse(name)).to eq('enyaW nhoJ')
    end
    ```

  * <a name="shared-examples"></a>
    Use shared examples to reduce code duplication.
    <sup>[[link](#shared-examples)]</sup>

    ```ruby
    # bad
    describe 'GET /articles' do
      let(:article) { FactoryBot.create(:article, owner: owner) }

      before { page.driver.get '/articles' }

      context 'when user is the owner' do
        let(:user) { owner }

        it 'shows all owned articles' do
          expect(page.status_code).to be(200)
          contains_resource resource
        end
      end

      context 'when user is an admin' do
        let(:user) { FactoryBot.create(:user, :admin) }

        it 'shows all resources' do
          expect(page.status_code).to be(200)
          contains_resource resource
        end
      end
    end

    # good
    describe 'GET /articles' do
      let(:article) { FactoryBot.create(:article, owner: owner) }

      before { page.driver.get '/articles' }

      shared_examples 'shows articles' do
        it 'shows all related articles' do
          expect(page.status_code).to be(200)
          contains_resource resource
        end
      end

      context 'when user is the owner' do
        let(:user) { owner }

        include_examples 'shows articles'
      end

      context 'when user is an admin' do
        let(:user) { FactoryBot.create(:user, :admin) }

        include_examples 'shows articles'
      end
    end

    # good
    describe 'GET /devices' do
      let(:resource) { FactoryBot.create(:device, created_from: user) }

      it_behaves_like 'a listable resource'
      it_behaves_like 'a paginable resource'
      it_behaves_like 'a searchable resource'
      it_behaves_like 'a filterable list'
    end
    ```

  * <a name="redundant-before-each"></a>
    Don't specify `:each`/`:example` scope for `before`/`after`/`around` blocks,
    as it is the default. Prefer `:example` when explicitly indicating the scope.
    <sup>[[link](#redundant-before-each)]</sup>

    ```ruby
    # bad
    describe '#summary' do
      before(:example) do
        # ...
      end

      # ...
    end

    # good
    describe '#summary' do
      before do
        # ...
      end

      # ...
    end
    ```

  * <a name="ambiguous-hook-scope"></a>
    Use `:context` instead of the ambiguous `:all` scope in `before`/`after`
    hooks.
    <sup>[[link](#ambiguous-hook-scope)]</sup>

    ```ruby
    # bad
    describe '#summary' do
      before(:all) do
        # ...
      end

      # ...
    end

    # good
    describe '#summary' do
      before(:context) do
        # ...
      end

      # ...
    end
    ```

  * <a name="avoid-hooks-with-context-scope"></a>
    Avoid using `before`/`after` with `:context` scope. Beware of the state
    leakage between the examples.
    <sup>[[link](#avoid-hooks-with-context-scope)]</sup>

## Example Structure

  * <a name="one-expectation"></a><a name="expectations-per-example"></a>
    For examples two styles are considered acceptable. The first variant
    is separate example for each expectation, which comes with a cost of
    repeated context initialization. The second variant is multiple
    expectations per example with `aggregate_failures` tag set for a
    group or example. Use your best judgement in each case, and apply
    your strategy consistently.
    <sup>[[link](#expectations-per-example)]</sup>

    ```ruby
    # good - one expectation per example
    describe ArticlesController do
      #...

      describe 'GET new' do
        it 'assigns a new article' do
          get :new
          expect(assigns[:article]).to be_a(Article)
        end

        it 'renders the new article template' do
          get :new
          expect(response).to render_template :new
        end
      end
    end

    # good - multiple expectations with aggregated failures
    describe ArticlesController do
      #...

      describe 'GET new', :aggregate_failures do
        it 'assigns new article and renders the new article template' do
          get :new
          expect(assigns[:article]).to be_a(Article)
          expect(response).to render_template :new
        end
      end

      # ...
    end
    ```

  * <a name="subject"></a>
    When several tests relate to the same subject, use `subject` to reduce
    repetition.
    <sup>[[link](#subject)]</sup>

    ```ruby
    # bad
    it { expect(hero.equipment).to be_heavy }
    it { expect(hero.equipment).to include 'sword' }

    # good
    subject(:equipment) { hero.equipment }

    it { expect(equipment).to be_heavy }
    it { expect(equipment).to include 'sword' }
    ```

  * <a name="use-subject"></a>
    Use named `subject` when possible. Only use anonymous subject declaration
    when you don't reference it in any tests, e.g. when `is_expected` is used.
    <sup>[[link](#use-subject)]</sup>

    ```ruby
    # bad
    describe Article do
      subject { FactoryBot.create(:article) }

      it 'is not published on creation' do
        expect(subject).not_to be_published
      end
    end

    # good
    describe Article do
      subject { FactoryBot.create(:article) }

      it 'is not published on creation' do
        is_expected.not_to be_published
      end
    end

    # even better
    describe Article do
      subject(:article) { FactoryBot.create(:article) }

      it 'is not published on creation' do
        expect(article).not_to be_published
      end
    end
    ```

  * <a name="subject-naming-in-context"></a>
    When you reassign subject with different attributes in different contexts, give
    different names to the subject, so it's easier to see what the actual subject
    represents.
    <sup>[[link](#subject-naming-in-context)]</sup>

    ```ruby
    # bad
    describe Article do
      context 'when there is an author' do
        subject(:article) { FactoryBot.create(:article, author: user) }

        it 'shows other articles by the same author' do
          expect(article.related_stories).to include(story1, story2)
        end
      end

      context 'when the author is anonymous' do
        subject(:article) { FactoryBot.create(:article, author: nil) }

        it 'matches stories by title' do
          expect(article.related_stories).to include(story3, story4)
        end
      end
    end

    # good
    describe Article do
      context 'when article has an author' do
        subject(:article) { FactoryBot.create(:article, author: user) }

        it 'shows other articles by the same author' do
          expect(article.related_stories).to include(story1, story2)
        end
      end

      context 'when the author is anonymous' do
        subject(:guest_article) { FactoryBot.create(:article, author: nil) }

        it 'matches stories by title' do
          expect(guest_article.related_stories).to include(story3, story4)
        end
      end
    end
    ```

  * <a name="dont-stub-subject"></a>
    Don't stub methods of the object under test, it's a code smell and
    often indicates a bad design of the object itself.
    <sup>[[link](#dont-stub-subject)]</sup>

    ```ruby
    # bad
    describe 'Article' do
      subject(:article) { Article.new }

      it 'indicates that the author is unknown' do
        allow(article).to receive(:author).and_return(nil)
        expect(article.description).to include('by an unknown author')
      end
    end

    # good - with correct subject initialization
    describe 'Article' do
      subject(:article) { Article.new(author: nil) }

      it 'indicates that the author is unknown' do
        expect(article.description).to include('by an unknown author')
      end
    end

    # good - with better object design
    describe 'Article' do
      subject(:presenter) { ArticlePresenter.new(article) }
      let(:article) { Article.new }

      it 'indicates that the author is unknown' do
        allow(article).to receive(:author).and_return(nil)
        expect(presenter.description).to include('by an unknown author')
      end
    end
    ```

  * <a name="it-and-specify"></a>
    Use `specify` if the example doesn't have a description, use `it` for
    examples with descriptions. An exception is one-line example, where
    `it` is preferable.
    `specify` is also useful when the docstring does not read well off
    of `it`.
    <sup>[[link](#it-and-specify)]</sup>

    ```ruby
    # bad
    it do
      # ...
    end

    specify 'it sends an email' do
      # ...
    end

    specify { is_expected.to be_truthy }

    it '#do_something is deprecated' do
      ...
    end

    # good
    specify do
      # ...
    end

    it 'sends an email' do
      # ...
    end

    it { is_expected.to be_truthy }

    specify '#do_something is deprecated' do
      ...
    end
    ```

  * <a name="it-in-iterators"></a>
    Do not write iterators to generate tests. When another developer adds a
    feature to one of the items in the iteration, they must then break it out into a
    separate test - they are forced to edit code that has nothing to do with their pull
    request.
    <sup>[[link](#it-in-iterators)]</sup>

    ```ruby
    # bad
    [:new, :show, :index].each do |action|
      it 'returns 200' do
        get action
        expect(response).to be_ok
      end
    end

    # good - more verbose, but better for the future development
    describe 'GET new' do
      it 'returns 200' do
        get :new
        expect(response).to be_ok
      end
    end

    describe 'GET show' do
      it 'returns 200' do
        get :show
        expect(response).to be_ok
      end
    end

    describe 'GET index' do
      it 'returns 200' do
        get :index
        expect(response).to be_ok
      end
    end
    ```

  * <a name="incidental-state"></a>
    Avoid incidental state as much as possible.
    <sup>[[link](#incidental-state)]</sup>

    ```ruby
    # bad
    it 'publishes the article' do
      article.publish

      # Creating another shared Article test object above would cause this
      # test to break
      expect(Article.count).to eq(2)
    end

    # good
    it 'publishes the article' do
      expect { article.publish }.to change(Article, :count).by(1)
    end
    ```

  * <a name="dry"></a>
    Be careful not to focus on being 'DRY' by moving repeated expectations into a
    shared environment too early, as this can lead to brittle tests that rely too
    much on one another.
    <sup>[[link](#dry)]</sup>

    It general it is best to start with doing everything directly in your `it`
    blocks even if it is duplication and then refactor your tests after you have
    them working to be a little more DRY. However, keep in mind that duplication
    in test suites is NOT frowned upon, in fact it is preferred if it provides
    easier understanding and reading of a test.

  * <a name="factories"></a>
    Use [Factory Bot](https://github.com/thoughtbot/factory_bot) to
    create test data in integration tests. You should very rarely
    have to use `ModelName.create` within an integration spec. Do
    **not** use fixtures as they are not nearly as maintainable as
    factories.
    <sup>[[link](#factories)]</sup>

    ```ruby
    # bad
    subject(:article) do
      Article.create(
        title: 'Piccolina',
        author: 'John Archer',
        published_at: '17 August 2172',
        approved: true
      )
    end

    # good
    subject(:article) { FactoryBot.create(:article) }
    ```

    *NOTE*: When talking about unit tests the best practice would be to
    use neither fixtures nor factories. Put as much of your domain logic
    in libraries that can be tested without needing complex, time
    consuming setup with either factories or fixtures.

  * <a name="needed-data"></a>
    Do not load more data than needed to test your code.
    <sup>[[link](#needed-data)]</sup>

    ```ruby
    # good
    RSpec.describe User do
      describe ".top" do
        subject { described_class.top(2) }

        before { FactoryBot.create_list(:user, 3) }

        it { is_expected.to have(2).items }
      end
    end
    ```

  * <a name="doubles"></a>
    Prefer using verifying doubles over normal doubles.
    <sup>[[link](#doubles)]</sup>

    Verifying doubles are a stricter alternative to normal doubles that
    provide guarantees, e.g. a failure will be triggered if an invalid
    method is being stubbed or a method is called with an invalid number
    of arguments.

    In general, use doubles with more isolated/behavioral tests rather
    than with integration tests.

    *NOTE*: There is no justification for turning
    `verify_partial_doubles` configuration option off. That will
    significantly reduce the confidence in partial doubles.

    ```ruby
    # good - verifying instance double
    article = instance_double('Article')
    allow(article).to receive(:author).and_return(nil)

    presenter = described_class.new(article)
    expect(presenter.title).to include('by an unknown author')


    # good - verifying object double
    article = object_double(Article.new, valid?: true)
    expect(article.save).to be true


    # good - verifying partial double
    allow(Article).to receive(:find).with(5).and_return(article)


    # good - verifying class double
    notifier = class_double('Notifier')
    expect(notifier).to receive(:notify).with('suspended as')
    ```

    *NOTE*: If you stub a method that could give a false-positive test result, you
    have gone too far.

  * <a name="dealing-with-time"></a>
    Always use [Timecop](https://github.com/travisjeffery/timecop) instead of
    stubbing anything on Time or Date.
    <sup>[[link](#dealing-with-time)]</sup>

    ```ruby
    # bad
    it 'offsets the time 2 days into the future' do
      current_time = Time.now
      allow(Time).to receive(:now).and_return(current_time)
      expect(subject.get_offset_time).to eq(current_time + 2.days)
    end

    # good
    it 'offsets the time 2 days into the future' do
      Timecop.freeze(Time.now) do
        expect(subject.get_offset_time).to eq 2.days.from_now
      end
    end
    ```

  * <a name="stub-http-requests"></a>
    Stub HTTP requests when the code is making them. Avoid hitting real
    external services.
    <sup>[[link](#stub-http-requests)]</sup>

    Use [webmock](https://github.com/bblimke/webmock) and
    [VCR](https://github.com/vcr/vcr) separately or
    [together](http://marnen.github.com/webmock-presentation/webmock.html).

    ```ruby
    # good
    context 'with unauthorized access' do
      let(:uri) { 'http://api.lelylan.com/types' }

      before { stub_request(:get, uri).to_return(status: 401, body: fixture('401.json')) }

      it 'returns access denied' do
        page.driver.get uri
        expect(page).to have_content 'Access denied'
      end
    end
    ```

## Naming

  * <a name="context-descriptions"></a>
    `context` block descriptions should always start with 'when', 'with',
    or 'without' and be in the form of a sentence with proper grammar when
    composed with `it` block descriptions.
    <sup>[[link](#context-descriptions)]</sup>

    ```ruby
    # bad
    context 'the display name not present' do
      # ...
    end

    context 'the display name has utf8 characters' do
      # ...
    end

    context 'the display name has no utf8 characters' do
      # ...
    end

    # good
    context 'when the display name is not present' do
      it 'raises an error' do
        # ...
      end
    end

    context 'with utf8 characters in the display name' do
      # ...
    end

    context 'without utf8 characters in the display name' do
      # ...
    end
    ```

  * <a name="example-descriptions"></a>
    `it`/`specify` block descriptions should never end with a conditional. This
    is a code smell that the `it` most likely needs to be wrapped in a `context`.
    <sup>[[link](#example-descriptions)]</sup>

    ```ruby
    # bad
    it 'returns the display name if it is present' do
      # ...
    end

    # good
    context 'when display name is present' do
      it 'returns the display name' do
        # ...
      end
    end

    # This encourages the addition of negative test cases that might have
    # been overlooked
    context 'when display name is not present' do
      it 'returns nil' do
        # ...
      end
    end
    ```

  * <a name="keep-example-descriptions-short"></a>
    Keep example description shorter than 60 characters.
    <sup>[[link](#keep-example-descriptions-short)]</sup>

    Write the example that documents itself, and generates proper
    documentation format output.

    ```ruby
    # bad
    it 'rewrites "should not return something" as "does not return something"' do
      # ...
    end

    # good
    it 'rewrites "should not return something"' do
      expect(rewrite('should not return something')).to
        eq 'does not return something'
    end

    # good - self-documenting
    specify do
      expect(rewrite('should not return something')).to
        eq 'does not return something'
    end
    ```

  * <a name="example-group-naming"></a>
    Prefix `describe` description with a hash for instance methods, with a
    dot for class methods.
    <sup>[[link](#example-group-naming)]</sup>

    Given the following exists

    ```ruby
    class Article
      def summary
        # ...
      end

      def self.latest
        # ...
      end
    end
    ```

    ```ruby
    # bad
    describe Article do
      describe 'summary' do
        #...
      end

      describe 'latest' do
        #...
      end
    end

    # good
    describe Article do
      describe '#summary' do
        #...
      end

      describe '.latest' do
        #...
      end
    end
    ```

  * <a name="should-in-it"></a>
    Do not write 'should' or 'should not' in the beginning of your
    example docstrings. The descriptions represent actual functionality
    - not what might be happening. Use the third person in the present
    tense.
    <sup>[[link](#should-in-it)]</sup>

    ```ruby
    # bad
    it 'should return the summary' do
      # ...
    end

    # good
    it 'returns the summary' do
      # ...
    end
    ```

  * <a name="describe-the-methods"></a>
    Be clear about what method you are describing. Use the Ruby
    documentation convention of `.` when referring to a class method's
    name and `#` when referring to an instance method's name.
    <sup>[[link](#describe-the-methods)]</sup>

    ```ruby
    # bad
    describe 'the authenticate method for User' do
      # ...
    end

    describe 'if the user is an admin' do
      # ...
    end

    # good
    describe '.authenticate' do
      # ...
    end

    describe '#admin?' do
      # ...
    end
    ```

  * <a name="use-expect"></a>
    On new projects always use the new `expect` syntax.
    <sup>[[link](#use-expect)]</sup>

    Configure RSpec to only accept the new `expect` syntax.

    ```ruby
    # bad
    it 'creates a resource' do
      response.should respond_with_content_type(:json)
    end

    # good
    it 'creates a resource' do
      expect(response).to respond_with_content_type(:json)
    end
    ```

## Matchers

  * <a name="predicate-matchers"></a>
    Use RSpec's predicate matcher methods when possible.
    <sup>[[link](#predicate-matchers)]</sup>

    ```ruby
    # bad
    it 'is published' do
      expect(subject.published?).to be true
    end

    # good
    it 'is published' do
      expect(subject).to be_published
    end
    ```

  * <a name="built-in-matchers"></a>
    Use built-in matchers.
    <sup>[[link](#built-in-matchers)]</sup>

    ```ruby
    # bad
    it 'includes a title' do
      expect(article.title.include?('a lengthy title')).to be true
    end

    # good
    it 'includes a title' do
      expect(article.title).to include 'a lengthy title'
    end
    ```

  * <a name="be-matcher"></a>
    Avoid using `be` matcher without arguments. It is too generic, as it pass on
    everything that is not `nil` or `false`. If that is the exact intend, use
    `be_truthy`. In all other cases it's better to specify what exactly is the
    expected value.
    <sup>[[link](#be-matcher)]</sup>

    ```ruby
    # bad
    it 'has author' do
      expect(article.author).to be
    end

    # good
    it 'has author' do
      expect(article.author).to be_truthy # same as the original
      expect(article.author).not_to be_nil # `be` is often used to check for non-nil value
      expect(article.author).to be_an(Author) # explicit check for the type of the value
    end
    ```

  * <a name="extract-common-expectation-parts-into-matchers"></a>
    Extract frequently used common logic from your examples into
    [custom matchers](https://relishapp.com/rspec/rspec-expectations/docs/custom-matchers/define-a-custom-matcher).
    <sup>[[link](#extract-common-expectation-parts-into-matchers)]</sup>

    ```ruby
    # bad
    it 'returns JSON with temperature in Celsius' do
      json = JSON.parse(response.body).with_indifferent_access
      expect(json[:celsius]).to eq 30
    end

    it 'returns JSON with temperature in Farenheit' do
      json = JSON.parse(response.body).with_indifferent_access
      expect(json[:farenheit]).to eq 86
    end

    # good
    it 'returns JSON with temperature in Celsius' do
      expect(response).to include_json(celsius: 30)
    end

    it 'returns JSON with temperature in Farenheit' do
      expect(response).to include_json(farenheit: 86)
    end
    ```

  * <a name="any_instance_of"></a>
    Avoid using `allow_any_instance_of`/`expect_any_instance_of`. It
    might be an indication that the object under test is too complex,
    and is ambiguous when used with receive counts.
    <sup>[[link](#any_instance_of)]</sup>

    ```ruby
    # bad
    it 'has a name' do
      allow_any_instance_of(User).to receive(:name).and_return('Tweedledee')
      expect(account.name).to eq 'Tweedledee'
    end

    # good
    let(:account) { Account.new(user) }

    it 'has a name' do
      allow(user).to receive(:name).and_return('Tweedledee')
      expect(account.name).to eq 'Tweedledee'
    end
    ```

## Rails

  * <a name="matcher-libraries"></a>
    Use third-party matcher libraries that provide convenience helpers
    that will significantly simplify the examples, [Shoulda Matchers](https://github.com/thoughtbot/shoulda-matchers)
    are one worth mentioning.
    <sup>[[link](#matcher-libraries)]</sup>

    ```ruby
    # bad
    describe '#title' do
      it 'is required' do
        article.title = nil
        article.valid?
        expect(article.errors[:title])
          .to contain_exactly('Article has no title')
        not
      end
    end

    # good
    describe '#title' do
      it 'is required' do
        expect(article).to validate_presence_of(:title)
          .with_message('Article has no title')
      end
    end
    ```

  * <a name="integration"></a>
    Test what you see.
    Deeply test your models and your application behaviour (integration
    tests). Do not add useless complexity testing controllers.
    <sup>[[link](#integration)]</sup>

    This is an open debate in the Ruby community and both sides have
    good arguments supporting their idea. People supporting the need of
    testing controllers will tell you that your integration tests don't
    cover all use cases and that they are slow. Both are wrong. It is
    possible to cover all use cases and it's possible to make them fast.

### Views

  * <a name="view-directory-structure"></a>
    The directory structure of the view specs `spec/views` matches the
    one in `app/views`. For example the specs for the views in
    `app/views/users` are placed in `spec/views/users`.
    <sup>[[link](#view-directory-structure)]</sup>

  * <a name="view-spec-file-name"></a>
    The naming convention for the view specs is adding `_spec.rb` to the
    view name, for example the view `_form.html.erb` has a
    corresponding spec `_form.html.erb_spec.rb`.
    <sup>[[link](#view-spec-file-name)]</sup>

  * <a name="view-outer-describe"></a>
    The outer `describe` block uses the path to the view without the
    `app/views` part. This is used by the `render` method when it is
    called without arguments.
    <sup>[[link](#view-outer-describe)]</sup>

    ```ruby
    # spec/views/articles/new.html.erb_spec.rb
    describe 'articles/new.html.erb' do
      # ...
    end
    ```

  * <a name="view-mock-models"></a>
    Always mock the models in the view specs. The purpose of the view is
    only to display information.
    <sup>[[link](#view-mock-models)]</sup>

  * <a name="view-assign"></a>
    The method `assign` supplies the instance variables which the view
    uses and are supplied by the controller.
    <sup>[[link](#view-assign)]</sup>

    ```ruby
    # spec/views/articles/edit.html.erb_spec.rb
    describe 'articles/edit.html.erb' do
      it 'renders the form for a new article creation' do
        assign(:article, double(Article).as_null_object)
        render
        expect(rendered).to have_selector('form',
          method: 'post',
          action: articles_path
        ) do |form|
          expect(form).to have_selector('input', type: 'submit')
        end
      end
    end
    ```

  * <a name="view-capybara-negative-selectors"></a>
    Prefer the capybara negative selectors over `to_not` with the positive.
    <sup>[[link](#view-capybara-negative-selectors)]</sup>

    ```ruby
    # bad
    expect(page).to_not have_selector('input', type: 'submit')
    expect(page).to_not have_xpath('tr')

    # good
    expect(page).to have_no_selector('input', type: 'submit')
    expect(page).to have_no_xpath('tr')
    ```

  * <a name="view-helper-stub"></a>
    When a view uses helper methods, these methods need to be stubbed. Stubbing
    the helper methods is done on the `template` object:
    <sup>[[link](#view-helper-stub)]</sup>

    ```ruby
    # app/helpers/articles_helper.rb
    class ArticlesHelper
      def formatted_date(date)
        # ...
      end
    end
    ```

    ```Rails
    # app/views/articles/show.html.erb
    <%= 'Published at: #{formatted_date(@article.published_at)}' %>
    ```

    ```ruby
    # spec/views/articles/show.html.erb_spec.rb
    describe 'articles/show.html.erb' do
      it 'displays the formatted date of article publishing' do
        article = double(Article, published_at: Date.new(2012, 01, 01))
        assign(:article, article)

        allow(template).to_receive(:formatted_date).with(article.published_at).and_return('01.01.2012')

        render
        expect(rendered).to have_content('Published at: 01.01.2012')
      end
    end
    ```

  * <a name="view-helpers"></a>
    The helpers specs are separated from the view specs in the `spec/helpers`
    directory.
    <sup>[[link](#view-helpers)]</sup>

### Controllers

  * <a name="controller-models"></a>
    Mock the models and stub their methods. Testing the controller should not
    depend on the model creation.
    <sup>[[link](#controller-models)]</sup>

  * <a name="controller-behaviour"></a>
    Test only the behaviour the controller should be responsible about:
    <sup>[[link](#controller-behaviour)]</sup>

      * Execution of particular methods
      * Data returned from the action - assigns, etc.
      * Result from the action - template render, redirect, etc.

    ```ruby
    # Example of a commonly used controller spec
    # spec/controllers/articles_controller_spec.rb
    # We are interested only in the actions the controller should perform
    # So we are mocking the model creation and stubbing its methods
    # And we concentrate only on the things the controller should do

    describe ArticlesController do
      # The model will be used in the specs for all methods of the controller
      let(:article) { double(Article) }

      describe 'POST create' do
        before { allow(Article).to receive(:new).and_return(article) }

        it 'creates a new article with the given attributes' do
          expect(Article).to receive(:new).with(title: 'The New Article Title').and_return(article)
          post :create, message: { title: 'The New Article Title' }
        end

        it 'saves the article' do
          expect(article).to receive(:save)
          post :create
        end

        it 'redirects to the Articles index' do
          allow(article).to receive(:save)
          post :create
          expect(response).to redirect_to(action: 'index')
        end
      end
    end
    ```

  * <a name="controller-contexts"></a>
    Use context when the controller action has different behaviour depending on
    the received params.
    <sup>[[link](#controller-contexts)]</sup>

    ```ruby
    # A classic example for use of contexts in a controller spec is creation or update when the object saves successfully or not.

    describe ArticlesController do
      let(:article) { double(Article) }

      describe 'POST create' do
        before { allow(Article).to receive(:new).and_return(article) }

        it 'creates a new article with the given attributes' do
          expect(Article).to receive(:new).with(title: 'The New Article Title').and_return(article)
          post :create, article: { title: 'The New Article Title' }
        end

        it 'saves the article' do
          expect(article).to receive(:save)
          post :create
        end

        context 'when the article saves successfully' do
          before do
            allow(article).to receive(:save).and_return(true)
          end

          it 'sets a flash[:notice] message' do
            post :create
            expect(flash[:notice]).to eq('The article was saved successfully.')
          end

          it 'redirects to the Articles index' do
            post :create
            expect(response).to redirect_to(action: 'index')
          end
        end

        context 'when the article fails to save' do
          before do
            allow(article).to receive(:save).and_return(false)
          end

          it 'assigns @article' do
            post :create
            expect(assigns[:article]).to eq(article)
          end

          it "re-renders the 'new' template" do
            post :create
            expect(response).to render_template('new')
          end
        end
      end
    end
    ```

### Models

  * <a name="model-mocks"></a>
    Do not mock the models in their own specs.
    <sup>[[link](#model-mocks)]</sup>

  * <a name="model-objects"></a>
    Use `FactoryBot.create` to make real objects, or just use a new (unsaved)
    instance with `subject`.
    <sup>[[link](#model-objects)]</sup>

    ```ruby
    describe Article do
      let(:article) { FactoryBot.create(:article) }

      # Currently, 'subject' is the same as 'Article.new'
      it 'is an instance of Article' do
        expect(subject).to be_an Article
      end

      it 'is not persisted' do
        expect(subject).to_not be_persisted
      end
    end
    ```

  * <a name="model-mock-associations"></a>
    It is acceptable to mock other models or child objects.
    <sup>[[link](#model-mock-associations)]</sup>

  * <a name="model-avoid-duplication"></a>
    Create the model for all examples in the spec to avoid duplication.
    <sup>[[link](#model-avoid-duplication)]</sup>

    ```ruby
    describe Article do
      let(:article) { FactoryBot.create(:article) }
    end
    ```

  * <a name="model-check-validity"></a>
    Add an example ensuring that the FactoryBot.created model is valid.
    <sup>[[link](#model-check-validity)]</sup>

    ```ruby
    describe Article do
      it 'is valid with valid attributes' do
        expect(article).to be_valid
      end
    end
    ```

  * <a name="model-validations"></a>
    When testing validations, use `expect(model.errors[:attribute].size).to eq(x)` to specify the attribute
    which should be validated. Using `be_valid` does not guarantee that the
    problem is in the intended attribute.
    <sup>[[link](#model-validations)]</sup>

    ```ruby
    # bad
    describe '#title' do
      it 'is required' do
        article.title = nil
        expect(article).to_not be_valid
      end
    end

    # preferred
    describe '#title' do
      it 'is required' do
        article.title = nil
        article.valid?
        expect(article.errors[:title].size).to eq(1)
      end
    end
    ```

  * <a name="model-separate-describe-for-attribute-validations"></a>
    Add a separate `describe` for each attribute which has validations.
    <sup>[[link](#model-separate-describe-for-attribute-validations)]</sup>

    ```ruby
    describe '#title' do
      it 'is required' do
        article.title = nil
        article.valid?
        expect(article.errors[:title].size).to eq(1)
      end
    end

    describe '#name' do
      it 'is required' do
        article.name = nil
        article.valid?
        expect(article.errors[:name].size).to eq(1)
      end
    end
    ```

  * <a name="model-name-another-object"></a>
    When testing uniqueness of a model attribute, name the other object
    `another_object`.
    <sup>[[link](#model-name-another-object)]</sup>

    ```ruby
    describe Article do
      describe '#title' do
        it 'is unique' do
          another_article = FactoryBot.create(:article, title: article.title)
          article.valid?
          expect(article.errors[:title].size).to eq(1)
        end
      end
    end
    ```

### Mailers

  * <a name="mailer-mock-model"></a>
    The model in the mailer spec should be mocked. The mailer should not depend on
    the model creation.
    <sup>[[link](#mailer-mock-model)]</sup>

  * <a name="mailer-expectations"></a>
    The mailer spec should verify that:
    <sup>[[link](#mailer-expectations)]</sup>

      * the subject is correct
      * the sender e-mail is correct
      * the e-mail is sent to the right e-mail address
      * the e-mail contains the required information

    ```ruby
    describe SubscriberMailer do
      let(:subscriber) { double(Subscription, email: 'johndoe@test.com', name: 'John Doe') }

      describe 'successful registration email' do
        subject { SubscriptionMailer.successful_registration_email(subscriber) }

        its(:subject) { should == 'Successful Registration!' }
        its(:from) { should == ['info@your_site.com'] }
        its(:to) { should == [subscriber.email] }

        it 'contains the subscriber name' do
          expect(subject.body.encoded).to match(subscriber.name)
        end
      end
    end
    ```

## Recommendations

  * <a name="correct-setup"></a>
    Correctly set up RSpec configuration globally (`~/.rspec`), per project
    (`.rspec`), and in project override file that is supposed to be kept out of
    version control (`.rspec-local`). Use `rspec --init` to generate `.rspec`
    and `spec/spec_helper.rb` files.
    <sup>[[link](#correct-setup)]</sup>

    ```
    # .rspec
    --color
    --require spec_helper

    # .rspec-local
    --profile 2
    ```

  * <a name="use-guard"></a>
    Running the whole suite on every change your can be very time
    consuming and may break the flow.
    [`guard-rspec`](https://github.com/guard/guard-rspec) continuously
    runs tests related to changed files only and may optionally notify
    if a failure happens.
    <sup>[[link](#use-guard)]</sup>

    Run `guard` in a separate shell.
    ```
    bundle exec guard
    ```

    Example `Guardfile`:

    ```
    guard 'rspec', cmd: 'bundle exec rspec' do
      # run every updated spec file
      watch(%r{^spec/.+_spec\.rb$})

      # run lib specs when a file in lib/ changes
      watch(%r{^lib/(.+)\.rb$}) { |_, name| "spec/lib/#{name}_spec.rb" }

      # run model specs related to the changed model
      watch(%r{^app/(.+)\.rb$}) { |_, name| "spec/#{name}_spec.rb" }

      # run view specs related to the changed view
      watch(%r{^app/(.*\.erb|haml))$}) { |_, name| "spec/#{name}_spec.rb" }

      # run integration specs related to the changed controller
      watch(%r{^app/controllers/(.+)\.rb}) { |_, name| "spec/requests/#{name}_spec.rb" }

      # run all integration tests when application controller change
      watch('app/controllers/application_controller.rb') { "spec/requests" }
    end
    ```

    *NOTE*: TDD workflow instead works best with a keybinding that runs
    just examples you want.

  * <a name="preloading"></a>
    To speed up initial loading time, preload the Rails app code with
    tools like [`spring`](https://github.com/rails/spring) and
    [`zeus`](https://github.com/burke/zeus).
    Those solutions will preload all gems that do not change unless
    `Gemfile` is updated, and reload models, view, factories etc if they
    are updated between test runs.
    <sup>[[link](#preloading)]</sup>

    *NOTE*: Those tools are often criticised to be a band aid on a
    problem that might be better solved through better design.

  * <a name="format"></a>
    Use a formatter that provides useful information about the test run
    and fits your workflow.
    [`fuubar`](https://github.com/thekompanee/fuubar) and
    [`fivemat`](https://github.com/tpope/fivemat) are two alternative
    formatters worth taking a look at.
    <sup>[[link](#format)]</sup>

    Add to the `Gemfile`:
    ```ruby
    group :test do
      gem 'fuubar'
    end
    ```

    `.rspec` global or project local configuration file:
    ```
    --format Fuubar
    ```

# Contributing

Nothing written in this guide is set in stone. Everyone is welcome to
contribute, so that we could ultimately create a resource that will be
beneficial to the entire Ruby community.

Feel free to open tickets or send pull requests with improvements.
Thanks in advance for your help!

You can also support the project (and RuboCop) with financial
contributions via [Patreon](https://www.patreon.com/bbatsov).

## How to Contribute?

It's easy, just follow the contribution guidelines below.

* [Fork](https://help.github.com/articles/fork-a-repo) the project on GitHub.
* Make your feature addition or bug fix in a feature branch.
* Include a [good description](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
of your changes.
* Push your feature branch to GitHub.
* Send a [Pull Request](https://help.github.com/articles/using-pull-requests).

# License

![Creative Commons License](http://i.creativecommons.org/l/by/3.0/88x31.png)
This work is licensed under a [Creative Commons Attribution 3.0 Unported License](http://creativecommons.org/licenses/by/3.0/deed.en_US)

# Credit

Inspiration was taken from the following:

[HowAboutWe's RSpec style guide](https://github.com/howaboutwe/rspec-style-guide)

[Community Rails style guide](https://github.com/rubocop-hq/rails-style-guide)

This guide was maintained by [ReachLocal](https://github.com/reachlocal)
for a long while.

This guide includes material originally present in
[BetterSpecs](https://github.com/lelylan/betterspecs)
([newer site](https://lelylan.github.io/betterspecs/) [older
site](http://www.betterspecs.org/)), sponsored by
[Lelylan](https://github.com/lelylan) and maintained by [Andrea
Reginato](https://github.com/andreareginato) and [many
others](https://github.com/lelylan/betterspecs/graphs/contributors) for a long while.
