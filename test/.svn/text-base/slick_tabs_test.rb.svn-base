require File.dirname(__FILE__) + '/../../../../test/test_helper'
require 'RMagick'

class SlickError < StandardError; end

class SlickTabsTest < Test::Unit::TestCase
  test_helper :hpricot_radiant
  test_helper :slick_tabs
  
  def setup
    @page = Page.new(:title => 'Test Page', :slug => "/")
    @page.id = 39
  end
     
  def teardown
    SlickTabsContextMgr.instance.destroy_all_contexts
  end
  
  def test_one_tab_thoroughly
    puts "***  thoroughly"

    path = File.join(RAILS_ROOT, 'public', 'images', 'slick_tabs', 'testthor-39.css')
    File.delete(path) if File.exist?(path)

    hp_setup(%Q|
    <r:slicktabs_header class="testthor"/>
      <r:slicktabs class="testthor">
        #{normal_style}
        <a href="jay-test">Jay Test</a>
      </r:slicktabs>|)

    css = element('link')
    assert_match %r|/images/slick_tabs/testthor-39.css?.*|, css.attributes['href']
    assert_equal 'text/css', css.attributes['type']
    assert_equal 'stylesheet', css.attributes['rel']
    assert File.exist?(path)
    
    ul = element('ul')
    assert_equal 2, ul.children.size # The final linefeed is the 2nd child

    li = ul.children.first
    assert_equal 1, li.children.size
    
    a = li.children.first
    assert_equal 'a', a.name
    assert_equal 'jay-test', a.attributes['href']
    assert_equal 2, a.children.size # The text of the label is the 2nd child
    
    span = a.children.first
    assert_equal 'span', span.name

#   FIXME parse the css file
#    assert_match %r|images/slick_tabs/Jay-Test-2184d2ec.png\?.*|, span.attributes['style']
    assert_equal 'Jay Test', a.inner_text

    path = File.join(RAILS_ROOT, 'public', 'images', 'slick_tabs', 'Jay-Test-3820809a.png')

    assert File.exist?(path)
    tab = Magick::Image.read(path).first

    assert_equal 108, tab.columns
    assert_equal 188, tab.rows    

  end
  
  def test_missing_style
    puts "***  missing style"
    assert_raise SlickError do
      hp_setup('<r:slicktabs_header/>
        <r:slicktabs>
          <a href="test">Test Tab</a>
        </r:slicktabs>')
    end

  end

  def test_missing_header_no_style
    puts "***  missing header no style"
    assert_raise SlickError do
      hp_setup('<r:slicktabs><a href="test">Test Tab</a></r:slicktabs>')  
    end

  end
  
  def test_missing_header_w_style
    puts "***  missing header with style"
    assert_raise SlickError do
      hp_setup(%Q|
        <r:slicktabs>
          #{normal_style}
          <a href="test">Test Tab</a>
        </r:slicktabs>|)  
    end
  end
  
  def test_slicktab_as_single
    puts "***  as single"
    assert_raise SlickError do
      hp_setup('<r:slicktabs_header/><r:slicktabs/>')
    end
  end
  
# TODO test_funky_char_label
#  def test_funky_char_label
#    assert_render_match(
#      %r{src="/images/nav_tabs/abc--easy-.*-trans\.png?.*"},
#      '<r:snazzytab label="abc! easy.as?*123"/>')
#  end

  def test_two_in_a_row
    puts "***  two in a row"  
    hp_setup(%Q|
    <r:slicktabs_header/>
    <r:slicktabs>
      #{normal_style}
      <a href="test1">Test One</a>
      <a href="test2">Test Two</a>
    </r:slicktabs>|)

    ul = element('ul')
    assert_equal 4, ul.children.size # LI, linefeed, LI, linefeed

    a = elements('a')[0]
    assert_equal a.inner_text, "Test One"
    assert_equal a.attributes['href'], "test1"

    span = elements('span')[0]
    regex = %r|images/slick_tabs/Test-One-(.*?)\.png\?.*\)|
#   FIXME parse the css file
#    assert_match regex, span.attributes['style']
#    m = regex.match(span.attributes['style'])
#    hash1 = m[1]

    a = elements('a')[1]
    assert_equal a.inner_text, "Test Two"
    assert_equal a.attributes['href'], "test2"

    span = elements('span')[1]
#   FIXME parse the css file
#    regex = %r|images/slick_tabs/Test-Two-(.*?)\.png\?.*\)|
#    m = regex.match(span.attributes['style'])
#    hash2 = m[1]
#
#    assert_not_equal hash1, hash2
#    assert_not_equal hash2, "0"
  end
  
  def test_wrong_class
    puts "***  wrong class"

    assert_raise SlickError do
      hp_setup(%Q|<r:slicktabs_header class="class1"/>
        <r:slicktabs class="class2">
          #{normal_style}
          <a href="test">Test Tab</a>
        </r:slicktabs>|)
      end
  end
  
  def test_missing_height
    puts "*** missing height"
    assert_raise SlickError do
      hp_setup(%Q|<r:slicktabs_header/>
        <r:slicktabs>
          <r:normal font_size="12"/>
          <a href="test">Test Tab</a>
        </r:slicktabs>|)
    end
  end

  def test_missing_size
    puts "*** missing size"
    assert_raise SlickError do
      hp_setup(%Q|<r:slicktabs_header/>
        <r:slicktabs>
          <r:normal height="94"/>
          <a href="test">Test Tab</a>
        </r:slicktabs>|)
    end
  end

end

#TODO test dir creation
#TODO test missing A-list
#TODO test two groups